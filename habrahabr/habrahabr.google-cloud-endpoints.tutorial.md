<h1> Введение </h1>
Google Cloud Endpoints – это надстройка над Google App Engine (GAE) для создания API для веб и мобильных приложений, делающая разработку проще и включающую в себя «из коробки» защиту от DoS-атак, OAuth 2.0 аторизацию, веб-интерфейс для тестирования API, SSL, атоматическую масштабируемость (сайт не упадет под хабра-эффектом), а также возможность использования сервисов доступных в Google App Engine (отсылка и прием электронной почты и XMPP-сообщений, загрузка данных из Интернет (URL Fetch service), задачи по расписанию (Task Queues and Scheduled Tasks) и др.)

GAE бесплатен в рамках начальных квот, которые позволяют попробовать и протестировать сервис, и также обеспечить бесплатное функционирование веб-сайта не имеющего больших нагрузок. При исчерпании квот сервис становиться платным.
Идея сервиса в том, что он делает всю или большую часть работы системного администратора, плюс некоторую часть работы программиста. Этот сервис может быть интересен стартапам, так как позволяет малыми силами и в котроткие сроки запустить рабочий проект.

Фреймворк <a href="https://github.com/objectify/objectify">Objectify</a> предоставляет удобные стредства для работы со базой данных встроенной в GAE, а модуль <a href="https://github.com/maximepvrt/angular-google-gapi">angular-google-gapi</a> для подключения веб-приложения на AngularJS.

Под катом много картинок и текста, и предполагается, что читатель знаком с Java Servlets.
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

в Linux (последняя версия 3.3.3):
<source lang="Bash">
sudo mkdir /usr/local/apache-maven/
cd /usr/local/apache-maven && sudo wget http://www.eu.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz && sudo tar xvf apache-maven-3.3.3-bin.tar.gz
echo 'export M2_HOME=/usr/local/apache-maven/apache-maven-3.3.3' >> ~/.bashrc
source ~/.bashrc
</source>
для Ubuntu/Debian (в депозитариях сейчас версия 3.0.5):
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

Define value for property 'version' - вводим номер версии, для GAE в номер версий должен быть без точек, например не 1.0.3, а 1-0-3. Это связанно с тем, что на GAE доступны версии проекта по адресу вида {номер версии}.{ID проекта}.appspot.com.
Также в GAE используется формат {номер instance}.{нормер версии}.{ID проекта}.appspot.com - поэтому рекомендуется имя версии начинать с буквы например v-1-0 или ver-1-0 , чтобы отличать от номера запущеной instance.

По этой причине вариант предлагаемый Maven по умолчанию - 1.0-SNAPSHOT - лучше заменять формат подходящий по стандартам GAE.

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

<code>your-app-id</code> следует заменить на {ID проекта}, в моем случае это hello-habrahabr-api (примечание: если вы не видите API в Services на .../_ah/api/explorer - возможно забыли заменить app.id)
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

При первом запуске Maven скачает Google App Engine SDK, и сохранит в
~/.m2/repository/com/google/appengine/appengine-java-sdk/{номер версии} (на Линукс)

Также при первом запуске из консоли нужно будет залогиниться в GAE, в окне браузера откроется диалог:

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.06.png"/>

при нажатии на кнопку, в новом окне появиться код, который нужно скопировать и ввести в консоли.

Токены аутентификации пользователя храняться в ~/.appcfg_oauth2_tokens_java
Если при попытке входа вы получаете сообщение 404 Not Found This application does not exist (app_id=u'{your-app-ID}') - это возможно связанно с тем, что вы были залогинены под другим эккаунтом. Переименуйте (или удалите) этот файл и попробуйте снова.

Если команда mvn appengine:update выполнена успешно ("BUILD SUCCESS") значит первоначальные настройки сделаны успешно.

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

На GAE есть возможность размещать бесплатный приватный Git репозиторий для своего кода. Репозиторий является только хранилищем, код из него не деплоится, на сервере выполняется .war файл который был собран и загружен с помощью Maven.

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

<i> 1.Установить Google Cloud SDK. </i>

Инструкции по установке на https://cloud.google.com/sdk/

Для Linux / Mac OS X :

<source lang="Bash">
curl https://sdk.cloud.google.com | bash # на вопросы можно принимать варианты предлагаемые по умолчанию
exec -l $SHELL
gcloud init # откроется окно браузера в котором надо залогиниться в учетную запись Google, потом в командной строке вводим имя нашего проекта
</source>
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.08.png"/>

В будущем настройки можно поменять командой: gcloud config

<source lang="Bash">gcloud config list</source> - показать настройки

<source lang="Bash">gcloud config set</source> - редактировать настройки

<source lang="Bash">gcloud config unset</source> - стереть настройки Google Cloud SDK

например:

gcloud config unset account - стереть учетную запись в настройках

gcloud config unset project - стереть имя проекта в настройках

<i> 2. Производим аутентификацию </i>

на Linux / Mac OS X :
<source lang="Bash">
gcloud auth login # откроется окно браузера в котором надо залогиниться в учетную запись Google
git config credential.helper gcloud.sh
</source>
<i> 2. добавляем удаленный git репозиторий GAE </i>

git remote add google https://source.developers.google.com/p/ {ID проекта} /

<source lang="Bash">
git remote add google https://source.developers.google.com/p/hello-habrahabr-api/
</source>
<i> 3. Выгружаем в удаленный репозиторий </i>
<source lang="Bash">
git push --all google
</source>
Чтобы клонировать репозиторий из облака GAE на локальную машину, нам потребуется:

Установить Google Cloud SDK как указано выше.

произвести аутентификацию:
<source lang="Bash">
gcloud auth login
</source>
клонировать репозиторий на локальную машину (инструкция на сайте предлагающая для этого gcloud init {ID проекта} - устарела):
<source lang="Bash">
gcloud source repos clone default {ID проекта}
</source>
переходим в созданную директорию, пишем и коммитим код в локальном репозитории, и
<source lang="Bash">
git push -u origin master
</source>
Для атоматизации работы можно использовать следующий скрипт (что-то вроде commit.push.build.and.upload.sh ):
<source lang="Bash">
mvn clean
git add * # предполагает наличие .gitignore
git commit -a
git push -u google
#
mvn install
mvn appengine:update
</source>
если приходится переключаться между проектами, в директорию проекта стоит разместить скрипт для быстрой и удобной смены настроек ( set.account.sh ):
<source lang="Bash">
gcloud config set account "{учетная запись @mgmail.com}"
gcloud config set project "{проект ID}"
gcloud config list
rm ~/.appcfg_oauth2_tokens_java
</source>
<h1> Приступаем к написанию API</h1>
Запускаем любимый редактор или IDE.

Для IntelliJ IDEA:

Для работы с GAE нам требуется Maven версии 3.1.0 и выше, в IntelliJ IDEA сейчас по умолчанию 3.0.5. При необходимости меняем, указывая директорию, в которой у нас установлена последняя версия Maven, например /usr/local/apache-maven/apache-maven-3.3.3 (проверить можно командой: echo $M2_HOME)

inport project -> указываем директорию проекта -> Import project from external model -> Maven -> можно оставить настройки по умолчанию -> Import Project -> указываем SDK (Java 1.7) -> Finish

Прежде всего рассмотрим как выглядит WEB-INF/web.xml. В нашем случае:
<source lang="XML">

<?xml version="1.0" encoding="utf-8" standalone="no"?>

<web-app
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.5"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <servlet>
        <servlet-name>SystemServiceServlet</servlet-name>
        <servlet-class>com.google.api.server.spi.SystemServiceServlet</servlet-class>
        <init-param>
            <param-name>services</param-name>
            <param-value>com.appspot.hello_habrahabr_api.YourFirstAPI</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>SystemServiceServlet</servlet-name>
        <url-pattern>/_ah/spi/*</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

</web-app>
</source>
Обратим внимание, что все запросы к API направляются по адресу: /_ah/spi/* и  обрабатываются сервлетом com.google.api.server.spi.SystemServiceServlet (SystemServiceServlet)

Одна из основных "фишек" Cloud Endpoints - веб-интерфейс для тестирования API (API Explorer) достутпен по адресу {проект ID}.appspot.com/_ah/api/explorer.

Для моделирования данных принимаемых и выдаваемых API используются JavaBean, т.е. классы отвечающие требованиям:

* публичный (public) конструктор без параметров. В данном случае конструктор должен быть указан в явном виде, хотя в примерах на сайте Google это упускают, но без конструктора на практике не работает.

* все свойства класса приватные, доступ через get/set (для boolean getter также должен начинаться с get, а не так как сгенерирует IDE)

* класс должен быть сериализуем (в явном виде можно не указывать)

Создадим два класса,

UserForm.java :
<source lang="Java">
package com.appspot.hello_habrahabr_api;

public class UserForm {

    private String name;
    private int age;
    private boolean ishuman;

    public UserForm() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public boolean getIshuman() {
        return ishuman;
    }
    // not just ishuman() for boolean
    // as will be created using getters and setters generation
    // in IntelliJ IDEA and Eclipse

    public void setIshuman(boolean ishuman) {
        this.ishuman = ishuman;
    }
}
</source>
и MessageToUser.java :
<source lang="Java">
package com.appspot.hello_habrahabr_api;

public class MessageToUser {

    private String name;
    private String message;
    private int usernumber;
    private boolean isregistered;

    public MessageToUser() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public int getUsernumber() {
        return usernumber;
    }

    public void setUsernumber(int usernumber) {
        this.usernumber = usernumber;
    }

    public boolean getIsregistered() {
        return isregistered;
    }
    // not "isregistered()" as suggested by IDE's getter/setter generator

    public void setIsregistered(boolean isregistered) {
        this.isregistered = isregistered;
    }
}

</source>

Теперь пишем класс для первого API-метода. Редактируем YourFirstApi.java , и вставим туда следующий код:
<source lang="Java">
package com.appspot.hello_habrahabr_api;

import com.google.api.server.spi.config.Api;
import com.google.api.server.spi.config.ApiMethod;
import com.google.api.server.spi.config.ApiMethod.HttpMethod;

@Api(
        name = "myApi",
        version = "v1",
        scopes = {Constants.EMAIL_SCOPE},
        description = "first API for this application."
)

public class YourFirstAPI {

    @ApiMethod(
            name = "register",
            path = "register",
            httpMethod = HttpMethod.POST
    )

    @SuppressWarnings("unused")

    public MessageToUser userInfo(
            final UserForm userForm
    ) {

        MessageToUser messageToUser = new MessageToUser();

        messageToUser.setMessage("Hi, " + userForm.getName() + ", you are registered on our site");

        messageToUser.setUsernumber(1);

        messageToUser.setIsregistered(true);

        return messageToUser;
    }
}
</source>

<h1> Веб-интерфейс к API (APIs Explorer) </h1>

теперь деплоим (mvn clean install && mvn appengine:update) и открываем в веб-браузере адрес https://{проект ID}.appspot.com/_ah/api/explorer , в моем случае https://hello-habrahabr-api.appspot.com/_ah/api/explorer
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_01.png"/>
Кликаем на название нашего API (если мы создадим несколько классов в аннотацией @Api - из будет несколько), и видим методы содержащиеся в этом API (методы класса с аннтотацией @ApiMethod ):

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_02.png"/>
Кликнув по полю "Request body" мы можем заполнить данные запроса получаемого api-методом:
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_03.png"/>

Далее мы можем выбрать "Autorize and execute" - и тогда нам потребуется пройти авторизацию ипользуя учетную запись Google (@ gmail.com) либо выбрать "Execute without OAuth", поскольку наше API пока никак не использует авторизацию мы увидим одинаковые результаты с авторизацией и без:
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_04.png"/>
"- Show headers -" в Response кликабельно.

<h1>Logging</h1>
Логи доступны по адресу: https://console.developers.google.com/project/{проект ID}/logs

Настройки в файле /src/main/webapp/WEB-INF/logging.properties и /src/main/webapp/WEB-INF/appengine-web.xml

Для того чтобы наш класс выдавал сообщения в лог, нужно классе API:

1.
<source lang="Java">
import java.util.logging.Logger;
</source>
2.
<source lang="Java">
@SuppressWarnings("unused")
private static final Logger LOG = Logger.getLogger({имя класса}.class.getName());
</source>

3. в метод добавим:
<source lang="Java">
LOG.info("сообщение");
</source>

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_09.png"/>

<h1> Авторизация OAuth 2.0 используя учетную запись Google (@ gmail.com)</h1>
В Constants.java добавим:
<source lang="Java">
import com.google.api.server.spi.Constant
</source>
и
<source lang="Java">
public static final String API_EXPLORER_CLIENT_ID = Constant.API_EXPLORER_CLIENT_ID
</source>
- это необходимо для тестирования OAuth-защищенных API-методов в APIs Explorer

Наш Constants.java будет выглядеть следующим образом:
<source lang="Java">
package com.appspot.hello_habrahabr_api;

import com.google.api.server.spi.Constant;

/**
 * Contains the client IDs and scopes for allowed clients consuming your API.
 */
public class Constants {
    public static final String WEB_CLIENT_ID = "replace this with your web client ID";
    public static final String ANDROID_CLIENT_ID = "replace this with your Android client ID";
    public static final String IOS_CLIENT_ID = "replace this with your iOS client ID";
    public static final String ANDROID_AUDIENCE = WEB_CLIENT_ID;
    public static final String EMAIL_SCOPE = "https://www.googleapis.com/auth/userinfo.email";

    public static final String API_EXPLORER_CLIENT_ID = Constant.API_EXPLORER_CLIENT_ID;
}
</source>

Теперь создадим новый класс :
<source lang="Java">

package com.appspot.hello_habrahabr_api;

import com.google.api.server.spi.config.Api;
import com.google.api.server.spi.config.ApiMethod;
import com.google.api.server.spi.config.ApiMethod.HttpMethod;
import com.google.api.server.spi.response.UnauthorizedException;
import com.google.appengine.api.users.User;
// https://cloud.google.com/appengine/docs/java/javadoc/index?com/google/appengine/api/users/User.html

import java.util.logging.Logger;

@Api(
        name = "oAuth2Api", // The api name must match '[a-z]+[A-Za-z0-9]*'
        version = "v1",
        description = "API using OAuth2"
)
public class OAuth2Api {


    @SuppressWarnings("unused")
    private static final Logger LOG = Logger.getLogger(OAuth2Api.class.getName());

    @ApiMethod(
            name = "getUserInfo",
            path = "getuserinfo",
            httpMethod = HttpMethod.POST
    )
    @SuppressWarnings("unused")
    public User getUserInfo(User user) throws UnauthorizedException {

        if (user == null) {
            LOG.warning("[warning] User not logged in");
            throw new UnauthorizedException("Authorization required");
        }

        return user;
    }
}
</source>
и пропишем его в init-param сервлета SystemServiceServlet в web.xml:
<source lang="XML">
    <servlet>
        <servlet-name>SystemServiceServlet</servlet-name>
        <servlet-class>com.google.api.server.spi.SystemServiceServlet</servlet-class>
        <init-param>
            <param-name>services</param-name>
            <param-value>
                com.appspot.hello_habrahabr_api.YourFirstAPI,
                com.appspot.hello_habrahabr_api.OAuth2Api
            </param-value>
        </init-param>
    </servlet>
</source>
деплоим проект, и смотрим API Explorer:
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_05.png"/>
Мы видим новое API в списке, кликнув по нему видим список его методов.
Кликаем на название метода:
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_07.png"/>
Теперь, если мы нажмем "Execute without OAuth" получим Exception:
<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/Selection_08.png"/>
Если кликаем "Autorize and execute" - нужно залогиниться используя учетную запись Google. В Response соотвественно получим email, nickname и userId (уникальный номер пользователя Google)

Объект класса com.google.appengine.api.users.User предоставляется GAE и содержит информацию о текущем пользователе, если пользователь не авторизован, соотвественно null. Таким образом мы можем проводить авторизацию используя логин-пароль учетной записи Google.

Как уже упоминалось на Хабре (<a href="http://habrahabr.ru/company/pushall/blog/268267/">Иногда лучше меньше — почему только Google-авторизация?</a>, <a href="http://habrahabr.ru/post/268253/">Юзабилити форм авторизации</a> ) проект вообще может обойтись без собственной обработки логинов и паролей.
На мой взляд это правильный подход, в первую очередь с точки зрения безопасности. Естественно, мы можем делать "регистрацию" на сайте после ввода дополнительной информации, платежа, и т.п.

<i> Создание фронтэнда на AngularJS рассмотрим в следующей статье. <i>
