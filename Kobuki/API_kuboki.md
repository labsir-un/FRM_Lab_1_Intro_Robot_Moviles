<div align="center">
<picture>
    <source srcset="https://imgur.com/5bYAzsb.png" media="(prefers-color-scheme: dark)">
    <source srcset="https://imgur.com/Os03JoE.png" media="(prefers-color-scheme: light)">
    <img src="https://imgur.com/Os03JoE.png" alt="Escudo UNAL" width="350px">
</picture>

<h3>Curso de Fundamentos de Robótica Móvil</h3>

<h1>Introducción a los Robots Móviles</h1>

<h2>Guía 1 - Kobuki: C++ API (Doxygen)</h2>

<h5>Pedro F. Cárdenas<br>
    Ricardo Ramírez<br>
    Juan S. Daleman</h5>

<h6>Universidad Nacional de Colombia<br>
    Facultad de Ingeniería<br>
    Departamento de Ingeniería Mecánica y Mecatrónica<br>
    Bogotá, Colombia<br>
    2025</h6>
</div>

<details>
    <summary>🗂️ Tabla de Contenido</summary>

<!-- TOC -->
- [1. 📖 Introducción](#1--introducción)
- [2. 🎯 Objetivos](#2--objetivos)
- [3. 🧰 Herramientas Necesarias](#3--herramientas-necesarias)
  - [3.1. 🔭🛠️ Equipos](#31-️-equipos)
  - [3.2. 🖥️💾 Software](#32-️-software)
- [4. 🔧➡️🚀 Procedimiento](#4-️-procedimiento)
  - [4.1. 🔧📦💻 Instalando de la API](#41--instalando-de-la-api)
  - [4.2. 🔌🚗🤖 Conectar robot Kobuki](#42--conectar-robot-kobuki)
  - [4.3. 📌⌨️🖥️ Comandos disponibles con la API](#43-️️-comandos-disponibles-con-la-api)
  - [4.4. ▶️🖥️ Ejecutar demos disponilbes con la API](#44-️️-ejecutar-demos-disponilbes-con-la-api)
- [5. 🤖⚔️ Retos](#5-️-retos)
- [6. 📚🗄️ Referencias](#6-️-referencias)
</details>

---

<h1> 🕵🏼🤖🚗 Guía 1: Uso de API para kobuki </h1>

## 1. 📖 Introducción

La API de Kobuki permite el control y la comunicación con la base móvil Kobuki, proporcionando herramientas para manejar sus motores, sensores y sistemas de energía. A través de esta API, los desarrolladores pueden enviar comandos de movimiento, leer datos de sensores como odometría y estado de la batería, e integrar el robot en sistemas más amplios mediante ROS u otras plataformas. Su diseño modular facilita la creación de aplicaciones personalizadas para navegación, teleoperación y automatización de tareas.

## 2. 🎯 Objetivos

- Introducir al uso de la API de Kuboki para su manejo desde C++.
- Mostar la ejecución de ejemplos que vienen con la API y la estructura de esta.

## 3. 🧰 Herramientas Necesarias

### 3.1. 🔭🛠️ Equipos

- Robot Kuboki.
- Computador.
- Cable USB-B a USB-A.

### 3.2. 🖥️💾 Software

- Ubuntu 20.04.
- C++ Versión c++14
- Compilador gcc
- Dependencias de compilación: ament, colcon, vcstool, CMake.
- Dependencias de código: Eigen, Sophus, ECL.

>[!NOTE]
>Para la descarga de dependencias se usa el comando `wget` por lo cual es necesario que tenga una conexión a internet.

## 4. 🔧➡️🚀 Procedimiento 

### 4.1. 🔧📦💻 Instalando de la API

1. Crea un directorio de trabajo.

```sh
cd ~/
mkdir kobuki && cd kobuki
```

2. Descarga el script para configurar el entorno virtual

```sh
wget https://raw.githubusercontent.com/kobuki-base/kobuki_documentation/release/1.0.x/resources/venv.bash || echo "Error: No se pudo descargar el archivo"
```

3. Descarga la configuración personalizada para la compilación.

```sh
wget https://raw.githubusercontent.com/kobuki-base/kobuki_documentation/release/1.0.x/resources/colcon.meta || echo "Error: No se pudo descargar el archivo"
```

4. Descarga la lista de repositorios de Kobuki.

```sh
wget https://raw.githubusercontent.com/kobuki-base/kobuki_documentation/release/1.0.x/resources/kobuki_standalone.repos || echo "Error: No se pudo descargar el archivo"
```

5. Obten las fuentes.

```sh
# Activar el entorno virtual
source ./venv.bash

# Crear directorio para el codigo fuente
mkdir src

# Descargar los repositorios desde el archivo .repos
vcs import ./src < kobuki_standalone.repos || echo "Error: No se pudo descargar los repositorios"

# Desactivar el entorno virtual
deactivate
```

6. Compila el código.

```sh
# Activar el entorno virtual
source ./venv.bash
```
```sh
# Compilar todo el proyecto
colcon build --merge-install --cmake-args -DBUILD_TESTING=OFF
```
```sh
# Actualizar los repositorios en src/
vcs pull ./src
```
```sh
# Desactivar el entorno virtual
deactivate
```

Otras opciones de compilación

```sh
# Compilar sin advertencias por variables no utilizadas
colcon build --merge-install --cmake-args -DBUILD_TESTING=OFF --no-warn-unused-cli
```
```sh
# Compilar un solo paquete. Ej: "kobuki_core"
colcon build --merge-install --packages-select kobuki_core --cmake-args -DBUILD_TESTING=OFF
```
```sh
# Compilar todo con salida detallada. Activa la salida detallada (VERBOSE=1) para depurar posibles errores
VERBOSE=1 colcon build --merge-install --event-handlers console_direct+ --cmake-args -DBUILD_TESTING=OFF
```
```sh
# Compilar en modo "Release" con símbolos de depuración
colcon build --merge-install --cmake-args -DBUILD_TESTING=OFF -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

Los encabezados, bibliotecas y recursos resultantes se pueden encontrar en ./install.

Estas instrucciones se verifican continuamente mediante una acción de GitHub  (yaml, results/logs).

### 4.2. 🔌🚗🤖 Conectar robot Kobuki

Kobuki se comunica principalmente a través de USB (también puede usar directamente un puerto serie, lo cual se explica más adelante). En la mayoría de los sistemas Linux, al conectar el cable, el robot aparecerá como un dispositivo en /dev/ttyUSB0.

Si tienes más de un dispositivo de este tipo conectado, Kobuki podría aparecer como /dev/ttyUSB1, /dev/ttyUSB2, etc.

Para asignarle un identificador constante, se puede usar una regla udev:

1. Descarga la regla udev.

```sh
wget https://raw.githubusercontent.com/kobuki-base/kobuki_ftdi/devel/60-kobuki.rules
```  

2. Copia la regla al sistema.

```sh
sudo cp 60-kobuki.rules /etc/udev/rules.d
```

3. Recargaa y reiniciaa udev.

```sh
sudo service udev reload
sudo service udev restart
```

4. Verifica la conexión.

```sh
#Confirma que el pueto que esta usando el kobuki
ls /dev/ | grep ttyUSB

#Verifica el funcionamiento de la regla /dev/kobuki
ls -l /dev/kobuki
```

### 4.3. 📌⌨️🖥️ Comandos disponibles con la API

Al usar el entorno de ejecución creado se habilitan diferentes comandos para usar la API con los cuales podemos ejecutar dos comandos `kobuki-version-info` y `kobuki-simple-keyop`.

1. Carga el entorno de ejecución

```sh
cd ~/kobuki
source ./install/setup.bash
```

2. Ejecuta `kobuki-version-info` para ver la información del kobuki.

3. Ejecuta `kobuki-simple-keyop` para operar el kuboki con el teclado.

>[!Note]
> Todos los comandos disponibles se encuentran en el directorio `~/kobuki/install/bin`

### 4.4. ▶️🖥️ Ejecutar demos disponilbes con la API

Con la API vienen con varios ejemplos de uso los cuales son:

- **demo_chirp:** Este ejemplo simplemente configura y establece una conexión con Kobuki, lo que hará que emita un sonido ("chirp"), haga una pausa de cinco segundos y luego reproduzca el sonido de apagado correspondiente.
- **demo_buttons:** Este ejemplo conecta un robot Kobuki y detecta eventos de botón. Cuando se suelta un botón, muestra una frase aleatoria en la consola.
- **demo_data_stream:** Este ejemplo se conecta a un robot Kobuki y escucha datos del flujo de sensores. Cada vez que recibe un nuevo paquete de datos, imprime los valores de los encoders de las ruedas en la consola.
- **demo_simple_loop:** Este ejemplo controla un robot Kobuki para moverse en un cuadrado de un tamaño determinado. Usa datos de odometría para actualizar su posición y ajusta su velocidad para avanzar en línea recta o girar 90° en cada esquina. Se ejecuta en un bucle continuo hasta que recibe una señal de interrupción (SIGINT).
- **demo_custom_logging:** Este ejemplo configura e inicializa un robot Kobuki, desactivando los registros predeterminados y conectando sus propias funciones de manejo de logs mediante señales y slots. Captura y muestra mensajes de depuración, información, advertencia y error con colores personalizados en la consola.
- **demo_raw_data_stream:** Este ejemplo en C++ captura y muestra en la consola los datos crudos recibidos del robot Kobuki en formato hexadecimal con marcas de tiempo y colores personalizados. Se conecta al flujo de datos mediante un sistema de señales y slots, inicializa el robot con el puerto de comunicación especificado y espera la recepción de paquetes.
- **demo_velocity_commands** Este ejemplo en C++ convierte comandos de velocidad lineal (vx) y angular (wz) en velocidades de rueda para un robot móvil. Implementa la clase Rb2Vw, que gestiona la conversión entre velocidades y radios de giro, asegurando un control preciso del movimiento. Además, incluye pruebas iterativas para validar la precisión de los cálculos y detectar posibles errores en la conversión de velocidades.

1. Carga el entorno de ejecución

```sh
cd ~/kobuki
source ./install/setup.bash
```

2. Entre en el directorio con los archivos compilados.

```sh
cd ~/kobuki/build/kobuki_core/src/demos
```

3. Para ejecutar algunos de los demos use la estructura ./<nombre_archivo>.

```sh
./demo_chirp
```

## 5. 🤖⚔️ Retos

1. Cree una rutina en la cual se use el sensor cliff para evitar caidas y cuando este detecte la caida genere un sonido.
2. Cree una rutina en la cual sensor de caída de rueda cuando se active cambie un led a rojo, al activarse los Bumpers cambie el led a naranja y cuando ningun sensor este activo el led sea de color verde.
3. En la rutina *"demo_simple_loop"* implemente un controlador el cual corrija el seguimiento del cuadrado propuesto.

*Extra:* Use *"Docking Station"* para una rutina propia.

## 6. 📚🗄️ Referencias

**[1]** Yujin Robot, "Kobuki - Software Overview", Kobuki Documentation (Development Branch), 2020. [Online]. Available: [(https://kobuki.readthedocs.io/en/devel/software.html](https://kobuki.readthedocs.io/en/devel/software.html)
