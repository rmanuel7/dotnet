# how to add docker to my local network windows
cómo acceder a una pagina web que esta en un contenedor docker desde otro pc

## Verificar el firewall de Windows:
El firewall de Windows podría estar bloqueando el tráfico entre los contenedores de Docker y la red local. Para asegurarte de que Docker puede comunicarse sin restricciones, sigue estos pasos:

### Habilitar reglas de tráfico en el firewall:
1. Abrir el Firewall de Windows:
   
   - Abre el **Panel de Control**.
     
   - Ve a **Sistema y seguridad** > **Firewall de Windows Defender**.
     
3. Permitir aplicaciones a través del firewall:
   
   - Haz clic en **Permitir una aplicación o una característica a través de Firewall de Windows Defender**.
     
     ![image](https://github.com/user-attachments/assets/d21c4796-4a43-4b7c-bc4d-ab5d9c8f7417)

   - Asegúrate de que **Docker Desktop** y las aplicaciones asociadas estén habilitadas para las redes **Privada** y **Pública**.
     
     ![image](https://github.com/user-attachments/assets/90d20973-704a-4a24-a955-e48ce1f28f6e)




