Active-Directory-attack-lab
Evaluación de seguridad de tipo caja negra sobre una infraestructura de laboratorio con máquinas Linux y Windows unidas a un dominio de Active Directory. El objetivo: partir de un acceso externo sin credenciales y llegar al control total del controlador de dominio (DC).
> ⚠️ Laboratorio propio en red aislada (sin acceso a internet). IPs, dominios, credenciales y hashes pertenecen a este entorno de pruebas y no corresponden a ningún sistema real.
Resumen ejecutivo
	
Tipo de prueba	Caja negra
Resultado	Control total del dominio (Golden Ticket)
Riesgo global	🔴 Crítico
Cadena de ataque	FTP anónimo → RCE web → credenciales en texto plano → pivoting → Kerberoasting → Golden Ticket
Durante la evaluación se logró comprometer un servidor Linux expuesto, obtener credenciales almacenadas en archivos del sistema, moverse lateralmente hacia una máquina Windows del dominio, extraer y crackear el hash de una cuenta de servicio vulnerable a Kerberoasting, y finalmente generar un Golden Ticket con el hash NTLM de `krbtgt`, obteniendo persistencia y privilegios de administrador de dominio.
Arquitectura
![Diagrama de red](images/01-diagrama-red.png)
Segmento	Rango	DHCP	Máquina
VLAN 100	10.0.100.0/24	Sí	Kali (atacante)
VLAN 110	10.0.110.0/24	Sí	Linux User
Host Only	192.168.56.0/24	No	Windows User
NAT	—	—	Windows DC
Herramientas empleadas
Nmap · Gobuster · Hashcat · PowerView · Mimikatz · sqlmap · crackmapexec · proxychains · xfreerdp3
Metodología
1. Reconocimiento
IP de la máquina atacante y descubrimiento de hosts en la red:
![ip a en Kali](images/02-ip-a-kali.png)
![arp-scan de la red](images/03-arp-scan.png)
Escaneo completo de puertos del host encontrado (10.0.100.5):
![Escaneo de puertos con nmap](images/04-nmap-puertos-10.0.100.5.png)
Puertos abiertos: 21 (FTP), 22 (SSH), 80 (HTTP).
2. Explotación del servidor web
Fuzzing de directorios con Gobuster:
![Gobuster fuzzing](images/05-gobuster-fuzzing.png)
Se descubre `/files/`, con un recurso `CALL.html`:
![Index of /files](images/06-index-of-files.png)
FTP con acceso anónimo habilitado — primer punto de entrada:
![Login FTP anónimo](images/07-ftp-anonymous-login.png)
Se sube una reverse shell en PHP al mismo directorio servido por Apache:
![Payload de reverse shell](images/08-payload-reverseshell-php.png)
![Subida del payload vía FTP](images/09-ftp-put-payload.png)
![Listado del payload subido](images/11-index-files-payload-subido.png)
Con un listener en escucha, se invoca el payload desde el navegador y se obtiene ejecución remota de código como `www-data`:
![Llamada al payload desde el navegador](images/12-browser-trigger-payload.png)
![Shell inversa obtenida](images/10-nc-listener-shell-obtenida.png)
3. Compromiso del sistema Linux
Se identifica la segunda interfaz de red de este host, puente hacia la siguiente subred:
![ip a como www-data](images/13-ip-a-linux-user.png)
Un archivo `important.txt` indica ejecutar un script que revela credenciales Linux y Windows en texto plano:
![Credenciales encontradas en /.runme.sh](images/14-credenciales-runme-sh.png)
El hash MD5 de las credenciales Linux se crackea con CrackStation:
![Crackeo del hash MD5](images/15-crackstation-crack-md5.png)
4. Pivoting de red
Con las credenciales Linux se establece un túnel SSH dinámico (SOCKS) para alcanzar la red interna donde vive la máquina Windows:
![Túnel dinámico SSH](images/16-ssh-tunel-dinamico.png)
Escaneo a través de `proxychains` para localizar la máquina Windows y sus puertos relevantes:
![Nmap vía proxychains (puertos clave)](images/17-proxychains-nmap-puertos.png)
![Nmap completo vía proxychains](images/18-proxychains-nmap-completo.png)
![Servicios Windows detectados](images/19-nmap-puertos-windows.png)
5. Acceso a la máquina Windows
Con las credenciales de dominio obtenidas, acceso remoto por RDP:
![Conexión RDP con xfreerdp3](images/20-xfreerdp-conexion.png)
6. Enumeración del dominio
```
ipconfig
```
![Configuración de red de Windows User](images/21-ipconfig-windows-user.png)
Localización del controlador de dominio con `crackmapexec`:
![Descubrimiento del DC con crackmapexec](images/22-crackmapexec-descubrimiento-dc.png)
Enumeración de cuentas de servicio con SPN (candidatas a Kerberoasting):
![Get-NetUser -SPN](images/23-getnetuser-spn.png)
7. Kerberoasting
Extracción del ticket/hash de la cuenta de servicio `HTTP/WINDOWS`:
![Obtención del hash con Get-DomainSPNTicket](images/24-getdomainspnticket-hash.png)
Crackeo offline con Hashcat (`-m 13100`):
![Hashcat crackea la contraseña del servicio IIS](images/25-hashcat-crack-iis-password.png)
La contraseña obtenida (`LaRosalia2021`) resulta válida también para autenticarse directamente en el dominio como esa cuenta de servicio:
![Autenticación exitosa como IIS_Service](images/26-crackmapexec-pwned-iis-service.png)
8. Escalada de privilegios — Golden Ticket
Se extraen los hashes NTDS del dominio, incluyendo el de la cuenta `krbtgt`:
![Dump de hashes NTDS](images/28-ntds-dump-hash-krbtgt.png)
Datos necesarios para construir el Golden Ticket, obtenidos con PowerView:
![Get-NetDomain](images/27-getnetdomain.png)
![Get-DomainSID](images/29-getdomainsid.png)
![Get-NetComputer](images/30-getnetcomputer-dc.png)
![Get-NetGroupMember "Domain Admins"](images/31-getnetgroupmember-domain-admins.png)
Usuario a suplantar: `Administrator` (RID 500):
![Administrator:500](images/32-administrator-500.png)
9. Persistencia
Generación del Golden Ticket con Mimikatz:
![Mimikatz](images/33-mimikatz-banner.png)
```
kerberos::golden /domain:example.com /sid:S-1-5-21-805668554-778713891-2534483124 \\\\
/rc4:610338dfc1b22a567b8f4377b031b13b /user:Administrator /id:500 /ptt
```
![Golden Ticket generado](images/34-kerberos-golden-ticket.png)
![Ticket cacheado (klist)](images/35-klist-tickets.png)
Acceso final al controlador de dominio como administrador:
![Enter-PSSession al DC — whoami](images/36-enter-pssession-dc-whoami.png)
Vulnerabilidades identificadas
ID	Vulnerabilidad	CWE	CVSS est.	Severidad
V1	Acceso FTP anónimo habilitado	CWE-284	7.5	Alto
V2	Permisos de escritura en directorio web	CWE-434	9.8	Crítico
V3	Credenciales almacenadas en archivos del sistema	CWE-522	9.0	Crítico
V4	Política de contraseñas débil	CWE-521	7.5	Alto
V5	Falta de segmentación de red	CWE-284	8.0	Alto
V6	Reutilización de credenciales entre sistemas	CWE-798	8.5	Alto
V7	Cuentas de servicio vulnerables a Kerberoasting	CWE-522	8.8	Alto
V8	Privilegios excesivos en cuentas de servicio	CWE-269	9.0	Crítico
V9	Exposición del hash de la cuenta krbtgt	CWE-522	10.0	Crítico
V10	Compromiso del controlador de dominio	CWE-284	10.0	Crítico
Recomendaciones clave
Deshabilitar el acceso FTP anónimo
Aplicar una política de contraseñas robusta
Eliminar credenciales almacenadas en archivos de texto plano
Segmentar adecuadamente las redes internas
Aplicar el principio de mínimo privilegio en cuentas técnicas
Proteger las cuentas de servicio (contraseñas largas y aleatorias, gMSA)
Rotar periódicamente la contraseña de `krbtgt`
Conclusión
El escenario reproduce un patrón habitual en incidentes reales: un servicio mal configurado expuesto a internet (FTP anónimo) es el punto de partida de una cadena que, combinando fallos individualmente moderados —credenciales en texto plano, contraseñas débiles, ausencia de segmentación, cuentas de servicio mal protegidas—, termina en el compromiso total de Active Directory. Ninguna de las vulnerabilidades por separado sería crítica; la cadena completa sí lo es.
---
Informe elaborado por Iván Salvador Santamaría · Laboratorio de práctica en entorno aislado.
