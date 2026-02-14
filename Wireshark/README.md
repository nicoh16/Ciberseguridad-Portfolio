# Análisis de tráfico con Wireshark

**Objetivo** 
Analizar y comparar el tráfico que llega a la máquina de la víctima utilizando *Nmap* de forma agresiva, y otra silenciosa. 

**Entorno**
Atacante: Kali Linux (IP: 10.0.2.15)
Víctima: Windows 7 vulnerable (IP 10.0.2.7)
Herramientas: Nmap (En Kali Linux) Y Wireshark (Windows 7)

**Metodología** 
1. Se inició captura con Wireshark en la máquina con Windows 7.
2. Se filtra la captura con `ip.addr == 10.0.2.7 && tcp`.
3. Se ejecutaron dos escaneos desde Kali: Uno agresivo, y otro más silencioso.
4. Se tomaron capturas de ambos por separado.

## Escenario de análisis
Se capturó el tráfico de una interfaz de red mientras se ejecutaba un escaneo **Nmap** contra un objetivo controlado. El objetivo fue identificar cómo se ven estas herramientas a nivel de capa de transporte (TCP).

### Escaneo agresivo:

Comando ejecutado desde Kali:
```bash
nmap -sV -A -T4 -p- 10.0.2.7
```

![Captura Wireshark con tráfico agresivo](./img/Captura-agresivo.png)

## Análisis de la captura:

* En esta captura se puede observar que los paquetes llegan con una diferencia de milisegundos, llegando a mas de 20 por segundo.
* El escaneo con -T4 y -A envía muchos paquetes SYN que son detectables por cualquier IDS/IPS al ser tan ruidoso.
* Resultado: Se prioriza obtener versiones y servicios, pero se detecta rápido.


### Escaneo silencioso:

Comando ejecutado desde Kali:
```bash
nmap -sS -T0 --scan-delay 10000ms -p 1-1000 10.0.2.7
```

## Puntos claves identificados:

### 1. Detección de TCP SYN Scan (Stealth Scan)
Se identificó el uso de la bandera `SYN` seguida de un `RST` (Reset) por parte del atacante.
- **Patrón:** `[SYN] -> [SYN, ACK] -> [RST]`.
- **Interpretación:** El atacante no completa el "Three-way handshake" para evitar ser detectado por aplicaciones simples.

### 2. OS Fingerprinting (Identificación del SO)
Analizando los paquetes de respuesta, se observaron los siguientes valores:
- **TTL (Time To Live):** Valores cercanos a 64 (sugiere Linux) o 128 (Sugiere Windows)
- **Window Size:** Utilizado para refinar la firma del sistema operativo remoto.

### 3. Filtro utilizado
Para aislar el ruido del resto de la red, se aplicó el siguiente filtro de visualización:
- `ip.addr == 10.0.2.7 && tcp` (Para centrar el análisis de la víctima)


## Evidencias

![Captura Wireshark con tráfico silencioso](./img/Captura-silenciosa.png)

## Análisis de la captura:

* Se puede ver que de esta forma los paquetes llegan con una frecuencia de 1 cada 10 segundos.
* Al usar -T0 se busca que se mezcle con el tráfico habitual de la red, intentando evadir los sistemas de monitoreo.
* Al reducir los paquetes enviados por segundo se pierde tiempo (Podría tomar horas obtener todos los puertos).
