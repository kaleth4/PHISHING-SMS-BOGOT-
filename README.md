Phishing Infrastructure Analysis (AWS S3 & CloudFront)Investigación técnica de una campaña activa de Smishing dirigida a usuarios en Colombia, utilizando técnicas de evasión mediante infraestructura distribuida en la nube.

📋 Resumen del IncidenteSe detectó un vector de ataque basado en ingeniería social (SMS) que redirige a las víctimas a un entorno de phishing altamente disponible alojado en AWS.Mensaje Detectado: "Su cuenta ha recibido 300.000 COP$. Por favor, revise su saldo lo antes posible." 

🛠️ Fase de Reconocimiento y RedirecciónEl atacante utiliza una cadena de saltos para dificultar el rastreo del servidor de origen.1. Análisis de Cabeceras (Redir Chain)Utilizando curl, se identificó una redirección permanente (301) desde un acortador hacia el dominio final.Bash# Comprobando el salto inicial
$ curl -I https://did.li/c5bsl
HTTP/2 301
location: https://www.xy321.club?tr=z34&r=18
server: AmazonS3
via: 1.1 CloudFront
2. Identificación del BackendEl análisis de las respuestas HTTP confirmó que el contenido es estático y servido a través de una red de distribución de contenido (CDN).Hosting: Amazon S3 (Bucket configurado para sitios web).CDN: CloudFront (Edge computing para baja latencia y evasión de IP-blocking).Caché: x-cache: Hit from cloudfront.

🔍 Fase de Enumeración (Scanning)Se realizaron pruebas de fuerza bruta de directorios para identificar archivos sensibles y configuraciones erróneas.Hallazgos con GobusterA pesar de las protecciones, se mapeó la siguiente estructura de respuesta:Plaintext403 Forbidden:  /.git/HEAD, /.ssh, /.bash_history, /.htpasswd
429 Too Many Requests: index.html, robots.txt, php.ini
404 Not Found: /index.html (Revela NoSuchKey de S3)
Evidencia de Configuración IncorrectaEl servidor expone detalles técnicos innecesarios en las páginas de error:XML<Error>
  <Code>NoSuchKey</Code>
  <Message>The specified key does not exist.</Message>
  <Key>/index.html</Key>
  <RequestId>MEAK921PV6DZAYM0</RequestId>
</Error>

🛡️ Controles de Seguridad DetectadosDurante el pentesting, se identificaron mecanismos de defensa activos en la infraestructura del atacante:Rate Limiting: El servidor bloquea ráfagas de peticiones con estados 429.Access Control: Restricción de acceso a archivos de configuración mediante estados 403.Cifrado: Implementación obligatoria de HTTPS vía redirección 301 desde el puerto 80.

📈 Conclusiones TécnicasSeveridad: ALTA. El uso de parámetros como ?tr=z34 sugiere un sistema de tracking para identificar individualmente a las víctimas.Infraestructura: El uso de AWS CloudFront permite al atacante rotar IPs rápidamente si los nodos son reportados.Vulnerabilidad Principal: Ingeniería social combinada con una configuración de infraestructura profesional que imita servicios legítimos.

d8021fd2f36dfc093593bee4e40fab268653058b4ac6964a8772ba6b980b07e1
www.xy321.club
