Google Cloud Endpoints на Java: руководство для начинающих
==========================================================

<h1> Введение </h1>
Google Cloud Endpoints – это надстройка над Google App Engine (GAE) для создания API для веб и мобильных приложений, делающая разработку проще и включающую в себя «из коробки» защиту от DoS-атак, OAuth 2.0 идентификацию, веб-интерфейс для тестирования API, SSL, а также возможность использования сервисов доступных в Google App Engine.

GAE бесплатен в рамках начальных квот, которые позволяют попробовать и протестировать сервис, и также обеспечить функционирование вебсайта без больших нагрузок. При исчерпании квот сервис становиться платным.
Идея сервиса в том, что он делает всю или большую часть работы системного администратора, плюс некоторую часть работы программиста.
Я предполагаю, что читатель знаком с Java и Servlets.

<h1> Регистрация </h1>
Для начала понадобится создать проект на GAE. Для этого потребуется учетная запись google. Можно воспользоваться существующей или создать новую. Политика Google допускает создание нескольких учетных записей для одного лица. Google предупреждает, когда приложение запрашивает данные пользователя (OAuth consent screen), то в числе данных о приложении может передаваться информация о разработчике, в том числе его адрес электронной почты, и это не любой адрес электронной почты, а именно учетной записи @ gmail.com под которой работают c GAE, так что возможно стоит заводить отдельные учетные записи для разных проектов, благо тот же Google Chrome позволяет удобно между ними переключаться.

Переходим в консоль разработчика Google: https://console.developers.google.com и логинимся используя выбранную учетную запись Google, нажимаем «Create an empty project», выбираем имя проекта и ID проекта. Проект будет доступен по адресу http://{ID проекта}.appspot.com

Например: hello-habrahabr-api.appspot.com

ID должно быть уникальным. При этом если {ID проекта}@ gmail.com занято, то этот ID уже будет отмечен как «занят» для GAE. То есть на получиться создать  {ID проекта }.appspot.com и {ID проекта}@ gmail.com одновременно.

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.01.png" />

После того как проект создан, он будет доступен в консоли разработчика:

<img src="https://raw.githubusercontent.com/ageyev/cloud-endpoints-tutorial/master/habrahabr/images/pic.02.png" />

Для данного руководства мы создадим два проекта в консоли разработчика Google:

hello-habrahabr-api.appspot.com , где будет API (собственно Google Cloud Endpoints), и hello-habrahabr-webapp.appspot.com , на котором разместим веб-клиент. Я делаю это для наглядности разделения фронэнда и бэкенда, можно было бы создавать на одном домене.

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

1.Java 7 (Java 8 пока не поддерживается GAE)

установка Java: http://www.java.com/en/download/manual_java7.jsp

для Ubuntu/Debian:

<source lang="Bash">
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update && sudo apt-get install oracle-java7-installer
</source>

2. Maven

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

3. IDE – по вкусу. Eclipse и IntelliJ IDEA Ultimate имеют плагины для работы с GAE, но тестирование оффлайн на своей машине для Cloud Endpoints все равно работать не будет (хотя в документации указано что должно).

Вместо GAE-плагинов проще и удобнее (имхо) использовать Maven в командной строке, так что вполне подойдет IntelliJ IDEA Community Edtion – там есть встроенный терминал, и это все что нужно. В Eclipse можно установить <a href="https://marketplace.eclipse.org/content/tm-terminal"> TM Terminal </a>

4. Google App Engine SDK. Можно скачать на https://cloud.google.com/appengine/downloads#Google_App_Engine_SDK_for_Java

<source lang="Bash">
mkdir ~/GAE-SDK && cd ~/GAE-SDK
wget https://storage.googleapis.com/appengine-sdks/featured/appengine-java-sdk-1.9.27.zip && unzip appengine-java-sdk-1.9.27.zip
</source>

















