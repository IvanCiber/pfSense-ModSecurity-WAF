# Análisis y diseño de una regla de detección SQL Injection con ModSecurity

## Descripción

Este ejercicio consiste en analizar y comprender una regla de seguridad de **ModSecurity** orientada a la detección de intentos de **SQL Injection (SQLi)** en peticiones HTTP.

El objetivo es identificar cómo funciona la regla, qué elementos de una petición inspecciona, qué patrones considera sospechosos y cómo puede contribuir a la detección y mitigación de ataques contra aplicaciones web.

## Objetivos

* Analizar la estructura y lógica de una regla de ModSecurity.
* Comprender los parámetros utilizados en la detección de SQL Injection.
* Identificar los elementos de una petición HTTP que son inspeccionados.
* Analizar las expresiones y patrones empleados para detectar comportamientos maliciosos.
* Comprender las acciones ejecutadas cuando se produce una coincidencia.
* Valorar las posibilidades y limitaciones de una regla de detección basada en patrones.

## Tecnologías y conceptos

* **ModSecurity**
* **OWASP Core Rule Set (CRS)**
* **SQL Injection (SQLi)**
* **HTTP/HTTPS**
* **Web Application Firewall (WAF)**
* **Expresiones regulares**
* **Reglas de detección**
* **Logs de seguridad**

## Análisis realizado

Durante el ejercicio se descompone la regla en sus diferentes componentes para comprender su funcionamiento:

1. **Condiciones de la regla**

   * Se analiza qué condiciones deben cumplirse para que se produzca una coincidencia.

2. **Elementos inspeccionados**

   * Se identifica qué partes de la petición HTTP son analizadas por ModSecurity.

3. **Patrones de detección**

   * Se estudian las expresiones y patrones utilizados para identificar posibles intentos de SQL Injection.

4. **Acciones**

   * Se analiza qué ocurre cuando la regla detecta una coincidencia y qué acciones puede ejecutar el WAF.

5. **Registro**

   * Se estudia cómo puede quedar registrada la detección para facilitar su posterior análisis desde una perspectiva SOC.

## Resultado

El ejercicio permite comprender el funcionamiento de una regla de detección de SQL Injection desde una perspectiva defensiva, relacionando la lógica de la regla con el comportamiento observado en una petición HTTP potencialmente maliciosa.

También permite entender la importancia de analizar una regla más allá de su sintaxis, identificando **qué detecta, por qué lo detecta y qué respuesta genera el sistema**.

## Estructura del repositorio

## Contexto

Este trabajo forma parte de un ejercicio práctico de **ciberseguridad defensiva**, centrado en el análisis de mecanismos de detección y protección de aplicaciones web mediante un WAF.

El objetivo no es únicamente identificar una coincidencia, sino desarrollar la capacidad de interpretar una regla de seguridad y relacionarla con el comportamiento de un posible ataque.

## Conclusiones

El análisis de reglas de ModSecurity permite adquirir una visión práctica de cómo un WAF puede detectar patrones asociados a ataques web y cómo estas detecciones pueden integrarse posteriormente en procesos de monitorización y respuesta dentro de un entorno SOC.
