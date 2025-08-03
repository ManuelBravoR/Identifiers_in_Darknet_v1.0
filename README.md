<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Documentación del Proyecto: Scraper de Identificadores en la Dark Web</title>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 900px;
            margin: 0 auto;
            padding: 2rem;
            background-color: #f9f9f9;
        }
        h1, h2, h3 {
            color: #2c3e50;
        }
        h1 {
            border-bottom: 2px solid #3498db;
            padding-bottom: 0.5rem;
        }
        h2 {
            border-bottom: 1px solid #bdc3c7;
            padding-bottom: 0.3rem;
            margin-top: 2rem;
        }
        h3 {
            margin-top: 1.5rem;
        }
        pre {
            background-color: #ecf0f1;
            padding: 1rem;
            border-radius: 5px;
            overflow-x: auto;
        }
        code {
            font-family: 'Courier New', Courier, monospace;
            background-color: #ecf0f1;
            padding: 2px 5px;
            border-radius: 3px;
        }
        pre code {
            padding: 0;
            background-color: transparent;
        }
        ul, ol {
            padding-left: 1.5rem;
        }
        a {
            color: #3498db;
            text-decoration: none;
        }
        a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>

    <h1>Documentación del Proyecto: Scraper de Identificadores en la Dark Web</h1>

    <h2>1. Objetivo del Proyecto</h2>
    <p>El objetivo de este proyecto es desarrollar y ejecutar una herramienta de scraping automatizada para la Dark Web. La finalidad es buscar identificadores de empresas (<code>identifiers</code>) que puedan estar expuestos en sitios <code>.onion</code>, lo que permite identificar posibles filtraciones de datos, fraudes, o comentarios ilícitos relacionados con las compañías.</p>

    <h2>2. Configuración del Entorno de Trabajo</h2>

    <h3>2.1. Instalación del Sistema Operativo</h3>
    <p>Aunque el proyecto puede ejecutarse en cualquier sistema, se utilizará <strong>Kali Linux</strong> por su practicidad en la instalación de herramientas de seguridad. Se recomienda la instalación en una Máquina Virtual (VM) con VMWare.</p>
    <ul>
        <li><strong>Enlace de descarga oficial de Kali Linux:</strong>
            <ul>
                <li><a href="https://www.kali.org/get-kali/#kali-platforms">https://www.kali.org/get-kali/#kali-platforms</a></li>
            </ul>
        </li>
        <li><strong>Enlace directo para VMWare:</strong>
            <ul>
                <li><a href="https://cdimage.kali.org/kali-2025.2/kali-linux-2025.2-vmware-amd64.7z">https://cdimage.kali.org/kali-2025.2/kali-linux-2025.2-vmware-amd64.7z</a></li>
            </ul>
        </li>
    </ul>
    <p>Una vez descargado y descomprimido, abra el archivo <code>.vmx</code> con VMWare.</p>
    <p><strong>Credenciales por defecto:</strong></p>
    <ul>
        <li><code>username</code>: <code>kali</code></li>
        <li><code>password</code>: <code>kali</code></li>
    </ul>

    <h3>2.2. Instalación de Tor y Dependencias</h3>
    <p>Los sitios con dominio <code>.onion</code> son inaccesibles a través de conexiones HTTP/HTTPS estándar. <strong>Tor</strong> es la puerta de enlace necesaria para resolver y conectar con estos servicios ocultos, permitiendo que nuestro scraper acceda a la Dark Web.</p>
    <p>Ejecute los siguientes comandos en la terminal de Kali:</p>
    <pre><code>sudo apt update
sudo apt list --upgradable
sudo apt install tor torsocks -y
sudo systemctl start tor
sudo systemctl enable tor
sudo systemctl status tor
</code></pre>
    <p><strong>Nota:</strong> Si surgen errores de firmas GPG (<code>NO_PUBKEY</code>), puede corregirlos ejecutando:</p>
    <pre><code>sudo rm /etc/apt/trusted.gpg.d/kali-archive-keyring.gpg
sudo wget -O /etc/apt/trusted.gpg.d/kali-archive-keyring.asc https://archive.kali.org/archive-key.asc
sudo apt update
</code></pre>

    <h3>2.3. Verificación de la Conexión a Tor</h3>
    <p>Para asegurarse de que Tor está funcionando y anonimizando su tráfico, puede usar <code>curl</code>. La salida debe mostrar una dirección IP diferente a la suya.</p>
    <ol>
        <li><strong>Verifique su IP pública sin Tor:</strong>
            <pre><code>curl ifconfig.me
</code></pre>
            <p>(Esto mostrará su IP real)</p>
        </li>
        <li><strong>Verifique su IP pública a través de Tor:</strong> Use <code>torsocks</code> para canalizar el tráfico a través del proxy SOCKS de Tor (puerto 9050).
            <ul>
                <li>Si obtiene un error <code>LD_PRELOAD</code>, ejecute el siguiente comando para corregirlo:
                    <pre><code>export LD_PRELOAD="/usr/lib/x86_64-linux-gnu/torsocks/libtorsocks.so"
</code></pre>
                </li>
            </ul>
            <p>Ahora, intente nuevamente:</p>
            <pre><code>torsocks curl ifconfig.me
</code></pre>
            <p>Si el comando devuelve una dirección IP diferente, la conexión a Tor funciona correctamente.</p>
        </li>
        <li><strong>Confirmación adicional con Tor Project:</strong>
            <pre><code>torsocks curl https://check.torproject.org/api/ip
</code></pre>
            <p>Una respuesta como <code>{"IsTor":true,"IP":"..."}</code> confirma que su tráfico está siendo enrutado a través de la red Tor.</p>
        </li>
    </ol>

    <h2>3. Configuración del Bot de Telegram</h2>
    <p>Para recibir notificaciones del scraping, implementaremos un bot de Telegram que enviará un reporte con las palabras clave encontradas.</p>

    <h3>3.1. Creación del Bot y Obtención del Token</h3>
    <ol>
        <li>Abra Telegram y busque a <strong><code>@BotFather</code></strong>.</li>
        <li>Use el comando <code>/newbot</code> y siga las instrucciones para crear su bot.</li>
        <li><strong><code>@BotFather</code></strong> le proporcionará un <strong><code>HTTP API Token</code></strong> que deberá guardar.</li>
    </ol>

    <h3>3.2. Obtención del Chat ID</h3>
    <p>Para que el bot sepa a quién enviar los mensajes, necesita su <strong><code>CHAT_ID</code></strong>.</p>
    <ol>
        <li>Busque a <strong><code>@userinfobot</code></strong> en Telegram y envíele un mensaje (<code>/start</code>).</li>
        <li>El bot le proporcionará su <code>ID</code> de usuario.</li>
    </ol>

    <h3>3.3. Creación del Fichero de Configuración</h3>
    <p>Cree un archivo llamado <code>telegram_config.py</code> en el directorio de su proyecto y añada su <code>BOT_TOKEN</code> y <code>CHAT_ID</code>.</p>
    <pre><code># telegram_config.py

BOT_TOKEN = 'tu_token_obtenido_de_BotFather'
CHAT_ID = 'tu_id_de_usuario'
</code></pre>

    <h2>4. Ejecución del Scraper de Python</h2>

    <h3>4.1. Preparación de Archivos</h3>
    <p>Dentro de su directorio de trabajo, <code>~/Documents/Scraping-onion</code>, necesitará dos archivos:</p>
    <ul>
        <li><code>identifiers.txt</code>: Un archivo de texto con las palabras clave o identificadores de empresa a buscar, uno por línea.</li>
        <li><code>seeds.txt</code>: Un archivo de texto con las URLs <code>.onion</code> de partida para el rastreo, una por línea. El scraper se ramificará a partir de estas semillas.</li>
    </ul>

    <h3>4.2. Preparación del Entorno Virtual y Dependencias</h3>
    <p>Para evitar conflictos de librerías, se utiliza un entorno virtual.</p>
    <pre><code># Obtenemos el script del repositorio
wget https://raw.githubusercontent.com/ManuelBravoR/Scraping-Onion-Sites/main/scraping_onion_sites.py

# Instalar venv si no está disponible
sudo apt install python3-venv -y

# Crear y activar el entorno virtual
python3 -m venv venv
source venv/bin/activate

# Instalar las librerías necesarias
pip install requests[socks] beautifulsoup4 requests-tor python-telegram-bot
</code></pre>
    <p>El <code>(venv)</code> en su terminal indica que el entorno virtual está activo.</p>

    <h3>4.3. Ejecución del Script</h3>
    <p>Para iniciar el scraper, ejecute el script desde la terminal, indicando el archivo de identificadores.</p>
    <pre><code>python3 scraping_onion_sites.py identifiers.txt
</code></pre>

    <h2>5. Resultados del Análisis</h2>
    <p>Al finalizar la ejecución, el script generará dos resultados principales:</p>
    <ul>
        <li><strong>Reporte en la Terminal:</strong>
            <p>La salida de la terminal mostrará el progreso del rastreo, las URLs visitadas y las palabras clave encontradas.</p>
        </li>
        <li><strong>Reporte en Telegram:</strong>
            <p>Se enviará un mensaje a su bot de Telegram con el contenido del archivo <code>report.txt</code>, el cual contiene un listado de las palabras clave encontradas y las URLs donde fueron identificadas.</p>
        </li>
    </ul>

    <h2>6. Mejoras y Continuidad</h2>
    <p>Esta sección detalla posibles mejoras para hacer el scraper más robusto, eficiente y autónomo.</p>

    <h3>6.1. Mejoras Continuas del Algoritmo</h3>
    <ul>
        <li><strong>Acceso a Contenido Protegido:</strong> El acceso a foros y sitios de mercados de datos a menudo requiere credenciales. Una mejora clave sería implementar la capacidad de iniciar sesión, lo que implica:
            <ul>
                <li>Bypass de CAPTCHA, ya sea con soluciones automáticas (como bibliotecas OCR o servicios de terceros).</li>
                <li>Creación y gestión de cuentas de usuario, almacenando las credenciales de forma segura.</li>
            </ul>
        </li>
        <li><strong>Rotación de Identidad:</strong> La actual configuración cambia de identidad Tor cada 5 requests. Para evitar bloqueos, se puede implementar una estrategia más avanzada de rotación de User-Agents y gestión de sesiones.</li>
    </ul>

    <h3>6.2. Automatización con Cronjob</h3>
    <p>La automatización es vital para un monitoreo continuo. Puede programar el script para que se ejecute automáticamente a intervalos regulares usando <code>cron</code>.</p>
    <p><strong>Ejecución en Horario Fijo:</strong></p>
    <p>Si el scraping siempre tarda menos de 10 minutos, puedes usar un <code>crontab</code> simple. El siguiente ejemplo ejecuta el script cada día a la 1:00 AM.</p>
    <pre><code>0 1 * * * cd /home/kali/Documents/Scraping-onion && /usr/bin/python3 scraping_onion_sites.py identifiers.txt
</code></pre>
    <p><strong>Ejecución Continuada (Recomendado):</strong></p>
    <p>Para evitar que se solapen los procesos, es mejor usar un script de shell que ejecute el scraper y espere un tiempo determinado antes de volver a empezar.</p>
    <ol>
        <li><strong>Crea el archivo <code>run_scraper.sh</code>:</strong>
            <pre><code>nano /home/kali/Documents/Scraping-onion/run_scraper.sh
</code></pre>
        </li>
        <li><strong>Añade este contenido y guarda:</strong>
            <pre><code>#!/bin/bash
while true
do
  echo "Iniciando el script de scraping..."
  python3 scraping_onion_sites.py identifiers.txt
  echo "Script finalizado. Esperando 10 minutos antes de la próxima ejecución..."
  sleep 600 # Espera 600 segundos (10 minutos)
done
</code></pre>
        </li>
        <li><strong>Haz el script ejecutable:</strong>
            <pre><code>chmod +x /home/kali/Documents/Scraping-onion/run_scraper.sh
</code></pre>
        </li>
        <li><strong>Configura <code>cron</code> para que lo inicie una vez:</strong>
            <pre><code>crontab -e
</code></pre>
            <p>Añade esta línea para que se ejecute al iniciar el sistema:</p>
            <pre><code>@reboot /home/kali/Documents/Scraping-onion/run_scraper.sh
</code></pre>
        </li>
    </ol>

    <h3>6.3. Otros Seeds Exitosos de Sitios .onion</h3>
    <p>Para obtener mejores resultados, es crucial partir de semillas relevantes (<code>seeds.txt</code>). A continuación, se presentan algunos ejemplos de sitios que pueden ser ricos en enlaces salientes y contenido:</p>
    <ul>
        <li><strong>Directorios y Wikis:</strong>
            <ul>
                <li><code>http://tor66sewebgixwhcqfnp5inzp5x5uohhdy3kvtnyfxc2e5mxiuh34iid.onion/</code></li>
                <li><code>http://darkfailenbsdla5mal2mxn2uz66od5vtzd5qozslagrfzachha3f3id.onion/</code></li>
                <li><code>http://zqktlwkvmv5ipqnik77wyxtb74bg6gtlwifjntdbanvprue7qqzaqlid.onion/wiki/index.php/Main_Page#Darknet_Marketplaces</code></li>
            </ul>
        </li>
        <li><strong>Servicios Financieros (Ejemplos de Temas):</strong>
            <ul>
                <li><code>http://imperialk4trdzxnpogppugbugvtce3yif62zsuyd2ag5y3fztlurwyd.onion/</code></li>
                <li><code>The PayPal Cent</code></li>
                <li><code>Bitcards</code></li>
                <li><code>PlasticSharks</code></li>
            </ul>
        </li>
    </ul>
    <p>El criterio para esta selección es que el sitio contenga otros sitios <code>.onion</code> o temas relevantes que el algoritmo pueda rastrear. Iniciar el rastreo desde un foro o un mercado de servicios financieros podría ofrecer resultados más específicos para el objetivo.</p>

</body>
</html>
