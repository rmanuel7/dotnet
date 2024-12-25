# Correlation failed.

An unhandled exception occurred while processing the request. Exception: Correlation failed.

# Cookies de sesión no persistentes:

Si las cookies no se persisten correctamente entre las solicitudes, la aplicación no puede correlacionar la `solicitud` de autenticación con la `respuesta`, lo que resulta en el error.

> [!NOTE]
> Cuando se trabaja con contenedores Docker, la configuración de las cookies de sesión puede no estar correctamente gestionada.


## Work with localhost
### `localhost` en Docker
Cuando usas `localhost` en la configuración del contenedor (dentro del contenedor mismo), las solicitudes se dirigen al contenedor que ejecuta la aplicación. Esto funciona porque dentro del contenedor, `localhost` hace referencia a él mismo, no a la máquina host.

![image](https://github.com/user-attachments/assets/386ed1bc-aaa1-4fd6-ba41-2ac192d7e53d)


## no work with host.docker.internal
### `192.168.1.2` y `host.docker.internal` no funcionan
1. `192.168.1.2`: La IP de tu máquina host desde el contenedor podría no ser accesible directamente debido a cómo Docker maneja las redes y el enrutamiento de tráfico.
2. `host.docker.internal`: Este es un *alias* especial de Docker para permitir que los contenedores se comuniquen con la máquina host. Sin embargo, no todos los entornos de Docker lo soportan (especialmente en Linux), y es posible que no funcione en tu configuración.

![image](https://github.com/user-attachments/assets/5ece3295-f443-409d-9900-3362dde0ee39)

![image](https://github.com/user-attachments/assets/9f8e1428-b98b-4177-bc18-0dce2c39bfc4)


> [!NOTE]
> `http://host.docker.internal:80`. Esto no se refiere al contenedor mismo, sino a la máquina que ejecuta Docker.

### Resumen de Diferencias
|Característica	|localhost	|host.docker.internal|
|---------------|-----------|--------------------|
**Dónde resuelve**|Resuelve a la IP local de la máquina o contenedor (127.0.0.1 en IPv4).|	Resuelve a la IP de la máquina host desde el contenedor.|
**En qué contexto funciona**|Se usa para hacer referencia a la misma máquina (contendor o host).	|Se usa para que un contenedor acceda a la máquina host desde dentro del contenedor.|
**Uso en contenedores Docker**|Se refiere al contenedor mismo, no al host.	|Se refiere a la máquina host desde dentro del contenedor.|
**Accesibilidad fuera del contenedor**|En un contenedor, `localhost` apunta al propio contenedor. En el host, apunta a la máquina local.	|`host.docker.internal` solo es válido dentro del contenedor Docker. No es accesible desde el host.|

1. **¿Cuándo Usar Cada Uno?**
- **Usa `localhost`**:
  - Cuando quieres hacer referencia al propio sistema (ya sea en el host o en el contenedor). Esto funcionará dentro del contenedor para acceder a servicios que están dentro del propio contenedor.
  - **Ejemplo**: Si tu contenedor tiene un servidor web corriendo dentro de él, y quieres acceder a él desde dentro del contenedor, usarás `localhost`.

- **Usa `host.docker.internal`**:
  - Cuando tu contenedor necesita acceder a un servicio que está corriendo en la máquina host (fuera del contenedor).
  - **Ejemplo**: Si tienes una base de datos corriendo en el host, pero tu aplicación está en un contenedor y necesitas acceder a esa base de datos, usarás `host.docker.internal`.
 
