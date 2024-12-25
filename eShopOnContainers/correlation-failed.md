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
