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
