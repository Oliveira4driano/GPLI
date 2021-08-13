# GPLI
Instalação do GLPi 9.5 no Debian 10

Vamos então de mão na massa!

Abaixo são dispostos os comandos comandos necessários para instalar o GLPi 9.5.x em um Servidor GNU/Linux Debian 10. Tentarei ser o mais descritivo possível!
Atualizando Lista de Pacotes Disponíveis

Nosso primeiro passo será atualizar a lista de pacotes disponíveis nos repositórios configurados no GNU/Linux Debian.
1
2
	
# Atualiza Lista de Pacotes
apt update

Com a lista de pacotes atualizada, podemos correr para os novos passos.
Fuso Horário

O GLPi 9.5 finalmente traz a possibilidade de podermos trabalhar com diferentes fusos na Central de Serviços.

Essa era uma funcionalidade há muito esperada por Centrais de Serviços de médio e grande porte que atendem Clientes geograficamente espalhados.

Mas há também um grande problema que vamos ajudar a resolver aqui!

Muitos analistas não configuram o servidor para trabalhar de forma confiável com relação a horários.

Aqui vão alguns comandos de ajuste geral para isso:
	
# Removendo pacotes NTP
apt purge ntp
 
# Instalar pacotes OpenNTPD
apt install -y openntpd
 
# Parando Serviço OpenNTPD
service openntpd stop
 
# Configurar Timezone padrão do Servidor
dpkg-reconfigure tzdata
 
 
# Adicionar servidor NTP.BR
echo "servers pool.ntp.br" > /etc/openntpd/ntpd.conf
 
# Habilitar e Iniciar Serviço OpenNTPD
systemctl enable openntpd
systemctl start openntpd

Importante selecionar o melhor Timezone para sua região!
![image-3](https://user-images.githubusercontent.com/33138839/129328625-030106b9-a4e1-43fd-b59b-90f797d61637.png)
![image-4](https://user-images.githubusercontent.com/33138839/129328838-28acda52-f679-4346-bd5c-6d4ccac97cc6.png)
Feito isso, teremos então um relógio “confiável” para trabalharmos!

Caso tenha interesse, acesse aqui o NTP.BR e conheça o projeto.
Pacotes para Manipulação de Arquivos e Outras Coisas

Se você tiver realizado uma instalação minimalista do sistema operacional, tal como sempre recomendamos, é provável que você precise de um pouco mais de ferramentas para manipulação de arquivos, consumir API dentre outras coisas.

Os comandos abaixo lhe auxiliarão com isso:
# PACOTES MANIPULAÇÃO DE ARQUIVOS
apt install -y xz-utils bsdtar bzip2 unzip curl

Feito isso, podemos seguir para os próximos passos!
Preparação do Servidor WEB

Como já sabemos, o GLPi trata-se de uma ferramenta WEB. Podemos vê-lo simplesmente como um site a ser instalado. Portanto, precisamos montar um ambiente WEB para tal funcionalidade.

Existem várias opções de serviço WEB a ser utilizada em ambientes GNU/Linux. Utilizaremos o servidor WEB Apache.

Para habilitar o serviço Apache em seu servidor, basta seguir o comando abaixo:

# Instalar dependências no sistema
apt install -y apache2 libapache2-mod-php7.3 php-soap php-cas php7.3 php7.3-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,bz2}

Repare que, ao invés de termos de ficar repetindo “php7.3-modulox” para cada módulo do PHP, usamos um recurso do shell usando todo conteúdo dentro do par de “chaves” ({ }). Isso cria um vetor com os valores dentro das chaves para que não tenhamos de ficar digitando tudo. Pois é, tem coisas que só o shell faz para nós!
O GLPi é um sistema desenvolvido na linguagem PHP, por isso, neste comando instalamos vários módulos PHP.

Ao fim deste comando, teremos um servidor WEB instalado já com suporte a linguagem PHP e com todas as dependências do GLPi resolvidas.
Resolvendo Problema de Acesso WEB ao Diretório

Um ajuste que muitos deixam de fazer é com relação a permissão de acesso ao diretório WEB. Isso pode ser resolvido de forma simples com uma pequeno arquivo de configuração.
![image-6](https://user-images.githubusercontent.com/33138839/129333433-5c112ec4-187c-49c6-950b-2de784fd966f.png)
Para criar o arquivo e habilitar a configuração, basta executar os comandos a seguir:

# Criar arquivo com conteúdo
echo -e "<Directory \"/var/www/html/glpi\">\nAllowOverride All\n</Directory>" > /etc/apache2/conf-available/glpi.conf
 
# Habilita a configuração criada
a2enconf glpi.conf
 
# Reinicia o servidor web considerando a nova configuração
systemctl restart apache2

Com isso, eliminamos um calo do nosso pé. Pode acreditar, muitos procuram a solução disso internet à fora!
Baixar e Instalar o GLPi

Como dito, o GLPi é basicamente um site. Nosso trabalho, neste momento é basicamente baixá-lo e colocá-lo em um diretório específico dentro do servidor. Neste caso, um diretório para onde tenha um site configurado no Apache.

Por comodidade, colocaremos o GLPi na pasta padrão do Apache, lembrando que isso não é uma regra.

Repare que fizemos tudo com apenas 1 linha de comando para você, de forma que fique bem simples:
	
# BAIXAR PACOTE GLPi
wget -O- https://github.com/glpi-project/glpi/releases/download/9.5.5/glpi-9.5.5.tgz | tar -zxv -C /var/www/html/

Estamos com apenas 1 linha de comando baixando o pacote GLPi de seu repositório com o comando “wget” e, já na sequência, usando o comando “tar” para extraí-lo direto no diretório do apache. 
# Ajustar Permissões de Arquivos
Com os arquivos já em seu devido lugar, faremos agora um ajuste em cima das permissões destes. Execute os seguintes comando:

# AJUSTAR PERMISSÕES DE ARQUIVOS
chown www-data. /var/www/html/glpi -Rf
find /var/www/html/glpi -type d -exec chmod 755 {} \;
find /var/www/html/glpi -type f -exec chmod 644 {} \;

Finalizado esta etapa

<p>Preparando o Serviço SQL</p>
# Instalando o Serviço MySQL
apt install -y mariadb-server
Ao fim deste comando teremos já um serviço MySQL disponível para ser utilizado em nosso Servidor.
Criando Usuário e Base de Dados MySQL

Apesar de termos nosso Serviço SQL rodando, ainda precisamos criar uma base de dados a ser utilizada pelo sistema e uma conta de acesso administrativo a essa base, também a ser utilizada pelo sistema GLPi.

Para tanto, basta usar os comandos abaixo:
	
# Criando base de dados
mysql -e "create database glpidb character set utf8"
 
# Criando usuário
mysql -e "create user 'verdanatech'@'localhost' identified by '123456'"
 
# Dando privilégios ao usuário
mysql -e "grant all privileges on glpidb.* to 'verdanatech'@'localhost' with grant option";

Mas relaxe! Já estamos chegando ao fim de nossa maratona. Apenas precisamos ajustar as permissões do usuário que acabamos de criar para acessar a tabela de TimeZone do MySQL. Lembra que agora podemos usar diferentes fusos no GLPi? É assim que habilitamos parte do processo!

Execute os comandos abaixo:
# Habilitando suporte ao timezone no MySQL/Mariadb
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
 
# Permitindo acesso do usuário ao TimeZone
mysql -e "GRANT SELECT ON mysql.time_zone_name TO 'verdanatech'@'localhost';"
 
# Forçando aplicação dos privilégios
mysql -e "FLUSH PRIVILEGES;"
Passos seguintes
Nas novas versões do GLPi, possuímos um pseudo prompt de comando desenvolvido pela equipe que nos permite executarmos várias coisas sem qualquer dor de cabeça e sem necessidade de acessarmos a interface WEB, inclusive.

Aqui, devemos tomar uma decisão:

    Realizar a instalação via Shell
    Realizar a instalação no modo padrão, via interface Web

Mostraremos os 2 modos para que você possa conhecê-los e escolher o melhor para seus laboratórios e implantações.

Instalação via Shell

Como dito antes, temos agora um pseudo prompt a ser utilizado. Trata-se de um arquivo em PHP chamado de “console” localizado em “/var/www/html/glpi/bin/”, segundo nossa instalação.

Para usá-lo, basta digitar seu caminho antecedido pelo comando ‘php’ e seguido dos parâmetros desejados que, em nosso caso, tratam-se dos parâmetros de instalação:
1
2
	
# Comando de instalação do Banco de Dados via Console
php /var/www/html/glpi/bin/console glpi:database:install --db-host=localhost --db-name=glpidb --db-user=verdanatech --db-password=123456

Repare que estamos passando todas as informações necessárias via linha de comando, tais como: Servidor sendo localhost, name da base de dados sendo glpidb, usuário sendo verdanatech e senha sendo 123456.
Uma observação que vale ouro é a de que, caso você utilize caracteres especiais em sua senha, proteja-a com aspas simples (--db-password='123@123') para inibir a interpretação pelo Shell.

![image-10-1024x164](https://user-images.githubusercontent.com/33138839/129336227-e5e478b9-c00a-4f2c-bdb9-612f431a521d.png)

Feito isso, seu GLPi estará pronto porém, existe uma pequena particularidade: os caches foram criados com permissões para o usuário root que executou a instalação e isso gerará um erro HTTP 500 ao tentar manipular estes arquivos do lado do navegador.

Para ajustar este problema, basta executar os comandos abaixo:
	
# Reajustando o acesso ao usuário do Apache
chown www-data. /var/www/html/glpi/files -Rf
Instalação via Web

Como de costume, podemos também realizar a instalação via navegador web. Basta abri-lo e acessar o endereço WEB do servidor GLPi que estávamos configurando e correr com a instalação tal como de costume.

Caso não tenha certeza de qual o endereço IP, basta digitar o seguinte comando no servidor:

hostname -I
![image-7](https://user-images.githubusercontent.com/33138839/129336426-17c48b58-ccd1-4e52-95df-5c0418d12dda.png)
Os passos a são relativamente intuitivos, com excessão da configuração da conexão com o banco de dados que alguns iniciantes se atrapalham. Mas aqui vai um print para ajudar:

