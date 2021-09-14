#Instrucciones

#App

  - Para crear la imagen de la app primero se debe realizar un {docker build -t <Nombre de la imagen que quieras>} en la misma carpeta donde este el Dockerfile y se creará la imagen, si haces esto recuerda que el docker-compose se tendra que editar.

#Docker-compose

  - Para utilizar el docker-compose, que te creará dos contenedores la app y base de datos persistente, solo se debe escribir {docker-compose up -d} y se generaran los dos contenedores anteriormente dichos. Si para crear la app quieres especificar el nombre de la imagen se debera seguir los pasos de arriba y editar el docker-compose quitando o comentando el {build: .} y descomentando el {image: (pon el nombre de la imagen)}.




#Info

  - El puerto que escucha la app es el 3000.

  - Para cambiar alguno de los textos de la app cuando esta en funcionamiento se pueden cambiar en: src/static/js/app.js y si quieres cambiar el titulo del principio está en: src/static/index.html
