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
