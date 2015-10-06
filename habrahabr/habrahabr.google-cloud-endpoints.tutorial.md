Google Cloud Endpoints на Java: руководство для начинающих
==========================================================

<h1> Введение </h1>
Google Cloud Endpoints – это надстройка над Google App Engine (GAE) для создания API для веб и мобильных приложений, делающая разработку проще и включающую в себя «из коробки» защиту от DoS-атак, OAuth 2.0 идентификацию, веб-интерфейс для тестирования API, SSL, атоматическая масштабируемость (сайт не упадет под хабра-эффектом), а также возможность использования сервисов доступных в Google App Engine (отсылка и прием электронной почты и XMPP-сообщений, загрузка данных из Интернет, задачи по расписанию и др.)

GAE бесплатен в рамках начальных квот, которые позволяют попробовать и протестировать сервис, и также обеспечить функционирование вебсайта без больших нагрузок. При исчерпании квот сервис становиться платным.
Идея сервиса в том, что он делает всю или большую часть работы системного администратора, плюс некоторую часть работы программиста. Этот сервис может быть интересен стартапам, так как позволяет малыми силами и в котроткие сроки запустить рабочий проект.

Под катом много картинок и текста, и предполагается, что читатель знаком с Java Servlets (для бэкенда) и AngularJS (для фронтенда).
<cut>

<h1> Регистрация </h1>
Для начала понадобится создать проект на GAE. Для этого потребуется учетная запись google. Можно воспользоваться существующей или создать новую. Политика Google допускает создание нескольких учетных записей для одного лица. Google предупреждает, когда приложение запрашивает данные пользователя (OAuth consent screen), то в числе данных о приложении может передаваться информация о разработчике, в том числе его адрес электронной почты, и это не любой адрес электронной почты, а именно учетной записи @ gmail.com под которой работают c GAE, так что возможно стоит заводить отдельные учетные записи для разных проектов, благо тот же Google Chrome позволяет удобно между ними переключаться.

Переходим в консоль разработчика Google: https://console.developers.google.com и логинимся используя выбранную учетную запись Google, нажимаем «Create an empty project», выбираем имя проекта и ID проекта. Проект будет доступен по адресу http://{ID проекта}.appspot.com

Например: hello-habrahabr-api.appspot.com

ID должно быть уникальным. При этом если {ID проекта}@ gmail.com занято, то этот ID уже будет отмечен как «занят» для GAE. То есть на получиться создать  {ID проекта}.appspot.com и {ID проекта}@ gmail.com одновременно.

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.01.png" />

После того как проект создан, он будет доступен в консоли разработчика:

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.02.png" />

Для данного руководства мы создадим два проекта в консоли разработчика Google: hello-habrahabr-api.appspot.com , где будет API (собственно Google Cloud Endpoints), и hello-habrahabr-webapp.appspot.com , на котором разместим веб-клиент. Я делаю это для наглядности разделения фронэнда и бэкенда, можно было бы создавать на одном домене.

В консоли разработчика нашего Cloud Endpoints проекта hello-habrahabr-api.appspot.com в меню выбираем APIs & auth > Credentials > Add Credentials > OAuth 2.0 Client ID > Configure consent screen

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.03.png" />

В настойках автоматически вставляется адрес электронной почты разработчика, также указываем имя приложения (Product name) (обязательно), и (опционально) адрес веб-страницы, логотип и другие данные.

В меню выбираем APIs & auth > Credentials > Add Credentials > OAuth 2.0 Client ID и указываем тип приложения для которого создаем client ID. В этом руководстве мы будем рассматривать создание веб-приложения, поэтому выбираем Web application, и нажимаем «Create»

Мы указываем:

- имя веб-приложения (любое)

- домен(ы) с которого Javascript может обращаться к API (Authorized JavaScript origins) это может быть любой домен. Можно тот же на котором работает Cloud Endpoints. При этом надо учитывать, что Cloud Endpoints пока не работает на домене пользователя, только на .appspot.com. Но веб-приложение которое обращается к API может работать на любом домене.

- Authorized redirect URIs — по умолчанию указываем тот же домент + /oauth2callback :  https://hello-habrahabr-webapp.appspot.com/oauth2callback

Нажимаем "Create", получаем "client ID" и "client secret" - они нам понадобятся для веб-приложения.

<h1> Подготовка и настройка рабочей среды.</h1>

Нам понадобится:

<i>1. Java 7</i> (Java 8 пока не поддерживается GAE)

установка Java: http://www.java.com/en/download/manual_java7.jsp

для Ubuntu/Debian:

<source lang="Bash">
sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update && sudo apt-get install oracle-java7-installer
</source>

<i>2. Maven</i>

установка: http://maven.apache.org/install.html

в Linux (последняя версия 3.3.3) :

<source lang="Bash">
sudo mkdir /usr/local/apache-maven/
cd /usr/local/apache-maven && sudo wget http://www.eu.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz && sudo tar xvf apache-maven-3.3.3-bin.tar.gz
echo 'export M2_HOME=/usr/local/apache-maven/apache-maven-3.3.3' >> ~/.bashrc
source ~/.bashrc
</source>

для Ubuntu/Debian (в депозитариях сейчас версия 3.0.5) :

<source lang="Bash">
sudo apt-get install maven
</source>

проверяем версию Maven и Java:

<source lang="Bash">
maven -version
</source>

<i>3. IDE или редактор кода</i> – по вкусу. Eclipse и IntelliJ IDEA Ultimate имеют плагины для работы с GAE, но тестирование оффлайн на своей машине для Cloud Endpoints все равно работать не будет (хотя в документации указано что должно), поэтому для разделения кода в разработке и работающей системы нужно либо использовать раздельные проекты, либо пользоваться предоставляемой GAE возможностью работы с разными версиями проекта.

Вместо GAE-плагинов проще и удобнее (имхо) использовать Maven в командной строке, так что вполне подойдет IntelliJ IDEA Community Edtion – там есть встроенный терминал, и это все что нужно. В Eclipse можно установить <a href="https://marketplace.eclipse.org/content/tm-terminal"> TM Terminal </a>

<i>4. Google App Engine SDK</i>. Если необходимо - можно скачать на https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Java

<source lang="Bash">
mkdir ~/GAE-SDK && cd ~/GAE-SDK
wget https://storage.googleapis.com/appengine-sdks/featured/appengine-java-sdk-1.9.27.zip && unzip appengine-java-sdk-1.9.27.zip
</source>

Но при использовании Maven, он скачает Google App Engine SDK автоматически.

<h1> Создаем основу бэкенда</h1>

Переходим в директорию в которой мы будем создавать директорию проекта. Избегайте директорий с нестандартными символами в наименовании и очень длинным путем, это может вызвать ошибки в Maven.

Для создания заготовки проекта используем Maven и <a href="http://mvnrepository.com/artifact/com.google.appengine.archetypes/endpoints-skeleton-archetype">Endpoints Skeleton Archetype</a> :

<source lang="Bash">
mvn archetype:generate -Dappengine-version=1.9.27 -Dfilter=com.google.appengine.archetypes:
</source>

вводим "2" (выбираем <code>2: remote -> com.google.appengine.archetypes:endpoints-skeleton-archetype (A skeleton project using Cloud Endpoints with Google App Engine Java)</code>)

выбираем версию (по умолчанию - последняя)

Define value for property 'groupId': - вводим groupId, это уникальный идентификатор нашего приложения, следуя <a href="https://maven.apache.org/guides/mini/guide-naming-conventions.html">Maven Guide to naming conventions on groupId, artifactId and version</a> - как правило используется имя домена в обратном порядке, но с учетом допустимых символов <a href="https://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html">package name rules</a> - то есть в моем случае вместо com.appspot.hello-habrahabr-api надо ввести com.appspot.hellohabrahabrapi или com.appspot.hello_habrahabr_api (я выбрал последнее)

Define value for property 'artifactId' - вводим краткое имя проекта, например hello-habrahabr-api (здесь дефис допускается). Так будет называться .war (.jar) файл без учета номера версии (имя файла будет hello-habrahabr-api-{номер версии}.war)

Define value for property 'version' - вводим номер версии, для GAE в номер версий должен быть без точек, например не 1.0.3, а 1-0-3. Это связанно с тем, что на GAE доступны версии проекта по адресу вида {номер версии}.{ID проекта}.
appspot.com.
Также в GAE используется формат {номер instance}.{нормер версии}{ID проекта}.
appspot.com - поэтому рекомендуется имя версии начинать с буквы например v-1-0 или ver-1-0 , чтобы отличать от номера запущеной instance.

По этой причине вариант предлагаемый Maven по умолчанию - 1.0-SNAPSHOT - нужно заменить.

Define value for property 'package' - обычно принимаем предложенное по умолчанию (будет соотвествовать groupId),

Подтверждаем выбранные настройки: Y (yes, по умолчанию)

Если все сделано правильно, получаем сообщение об успешной сборке:

<source lang="Bash">
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 03:08 min
[INFO] Finished at: 2015-10-06T02:42:33+03:00
[INFO] Final Memory: 16M/108M
[INFO] ------------------------------------------------------------------------
</source>

Maven создаст папку имя которой будет соотвествовать artifactId с файлами проекта. Папка имеет следующую структуру:

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.05.png"/>

<h2>pom.xml</h2>
Для начала нам следует отредактировать pom.xml который содержит настройки Maven для нашего проекта. В

<source lang="XML">
<properties>
    <app.id>your-app-id</app.id>
    <app.version>1</app.version>

</properties>
</source>

<code>your-app-id</code> следует заменить на {ID проекта}, в моем случае это hello-habrahabr-api
и <app.version>ver-1-0</app.version> приводим в соотвествие с <version>ver-1-0</version>

в

<source lang="XML">
<prerequisites>
    <maven>3.1.0</maven>
</prerequisites>
</source>

прописываем ту версию Maven, которая у нас установлена, в моем случае 3.1.0 я заменяю на 3.3.3

<h2>src/main/webapp/WEB-INF/appengine-web.xml</h2>

В src/main/webapp/WEB-INF/appengine-web.xml

<source lang="XML">
<application>${app.id}</application>
<version>${app.version}</version>
</source>

{app.id} и {app.version} заменяем соотвественно на {ID проекта} и версию.
(нужно заменить, автоматически из pom.xml оно значения не возьмет)

собираем проект:
<source lang="Bash">
mvn clean install
</source>

Если сборка была неуспешной, ищем ошибки в настройках. Если - успешной, загружаем проект на GAE:

<source lang="Bash">
mvn appengine:update
</source>

При первом запуске Maven скачает Google App Engine SDK, и сохрани (на Линукс) в
~/.m2/repository/com/google/appengine/appengine-java-sdk/{номер версии}

Также при первом запуске из консоли нужно будет залогиниться в GAE, в окне браузера откроется диалог:

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.06.png"/>

при нажатии на кнопку, в новом окне появиться код, который нужно скопировать и ввести в консоли.

Токены аутентификации пользователя храняться в ~/.appcfg_oauth2_tokens_java
Если при попытке входа вы получаете сообщение 404 Not Found This application does not exist (app_id=u'{your-app-ID}') - это возможно связанно с тем, что вы были залогинены под другим эккаунтом. Переименуйте (или удалите) этот файл и попробуйте снова.

Если команда mvn appengine:update выполнена успешно ("BUILD SUCCESS
") значит первоначальные настройки сделаны успешно.

Наблюдался баг при загрузке в версии 1.9.21, который можно было исправить следующим образом: в pom.xml во всех местна заменяем appengine.version на appengine.sdk.version (см. https://code.google.com/p/googleappengine/issues/detail?id=12009 ) В текущей верси (1.9.27) баг не проявляется, но знать на всякий случай не помешает.

В версии 1.9.27 при загрузке можно получить сообщение:

<source lang="Bash">
********************************************************
The API version in this SDK is no longer supported on the server!
-----------
Latest SDK:
Release: 1.9.27
Timestamp: Tue Aug 18 21:28:24 IDT 2015
API versions: [1]

-----------
Your SDK:
Release: 1.9.27
Timestamp: Wed Sep 02 23:29:33 IDT 2015
API versions: [1.0]

-----------
Please visit https://developers.google.com/appengine/downloads for the latest SDK.
********************************************************
</source>

Несмотря на это предупреждение, загрузка должна проходить успешно.

<h1> Настраиваем Git на GAE </h1>

На GAE есть возможность размещать бесплатный приватный Git репозиторий для своего кода. При просмотре логов можно получить ссылку на соотвествущий файл в репозитории, что удобно при тестировании и устранении ошибок.

Сначала инициализируем git репозиторий в директории проекта:

<source lang="Bash">
git init
git add src/ pom.xml README.md
git commit -a
</source>

Переходим в консоль разработчика: https://console.developers.google.com/project, и выбираем наш проект.

Выбираем в меню: Source code > Browse

Сайт предложит создать репозиторий Git. Жмем "Get started"

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.07.png"/>

Можно скопировать код с существующего репозитория на Github или Bitbucket, загрузить (push) код со своего копьютера, или клонировать (clone) код проекта с GAE на свой компьютер.

Выберем "push". Получаем инструкцию:

<h3> 1.Установить Google Cloud SDK. </h3>

Инструкции по установке на https://cloud.google.com/sdk/

Для Linux / Mac OS X :

<source lang="Bash">

curl https://sdk.cloud.google.com | bash # на вопросы можно принимать варианты предлагаемые по умолчанию

exec -l $SHELL

gcloud init # откроется окно браузера в котором надо залогиниться в учетную запись Google, потом в командной строке вводим имя нашего проекта
</source>

В будущем настройки можно поменять командой: gcloud config

gcloud config list - показать настройки

gcloud config set - редактировать настройки

gcloud config unset - стереть настройки Google Cloud SDK

<h3> 2. Производим аутентификацию </h3>

на Linux / Mac OS X :

<source lang="Bash">
gcloud auth login # откроется окно браузера в котором надо залогиниться в учетную запись Google
git config credential.helper gcloud.sh
</source>

<h3> 2. добавляем удаленный git репозиторий GAE </h3>

git remote add google https://source.developers.google.com/p/{ID проекта}/

<source lang="Bash">
git remote add google https://source.developers.google.com/p/hello-habrahabr-api/
</source>

<h3> Выгружаем в удаленный репозиторий </h3>

<source lang="Bash">
git push --all google
</source>

Чтобы клонировать репозиторий из облака GAE на локальную машину, нам потребуется:

Установить Google Cloud SDK как указано выше.

произвести аутентификацию:

<source lang="Bash">
gcloud auth login
</source>

клонировать репозиторий на локальную машину:

<source lang="Bash">
gcloud init {проект ID (hello-habrahabr-api)}
</source>

переходим в созданную директорию:

<source lang="Bash">
cd {проект ID (hello-habrahabr-api)}/default
</source>

Пишем и коммитим код в локальном репозитории, и

<source lang="Bash">
git push -u origin master
</source>

См. также:
https://cloud.google.com/tools/repo/push-to-deploy




http://hello-habrahabr-api.appspot.com/_ah/api/explorer

