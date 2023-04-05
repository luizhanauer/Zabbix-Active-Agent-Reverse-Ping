Configuração de Agente Ativo do Zabbix para Ping Reverso
========================================================

Este guia contém todos os passos necessários para configurar um agente ativo no Zabbix para coletar a latência de ping reverso de um host. Além disso, inclui os passos para configurar um gráfico e uma trigger para o item de ping reverso.

Pré-requisitos
--------------

*   Um servidor Zabbix configurado e em execução.
*   Acesso de root ou sudo no servidor Zabbix e no host que será monitorado.
*   O pacote `fping` instalado no host que será monitorado.

Passo 1: Configurar o Agente Ativo do Zabbix
--------------------------------------------

O primeiro passo é configurar o agente ativo do Zabbix para coletar a latência de ping reverso. Para fazer isso, execute os seguintes comandos no host que será monitorado:

Para ajustar as configurações do agente do Zabbix, acesse o arquivo `/etc/zabbix/zabbix_agentd.conf`. Nele poderá fazer ajustes em demais parâmetros. Você pode usar o ´nano´, ´vim´ ou outro de sua preferencia para editar o arquivo.

Será nescessario que ao menos os parâmetros a baixo estejam configurados:
* Server - endereço do host IP ou DNS do Zabbix Server (Quando estiver como agente ativo pode deixar esse parâmetro em branco)
* ServerActive - endereço do host IP ou DNS do Zabbix Server
* Hostname - nome ao qual será identificado as coletas no zabbix server
* UnsafeUserParameters - descomentar e trocar de 0 para o valor 1

```
nano /etc/zabbix/zabbix_agentd.conf
```
&nbsp;
Se sua instalação estiver limpa, você pode aplicar diretamente os comandos a baixo (alterando conforme sua nescessidade).

* Deixar o parâmetro ´Server´ em branco.
```
sed -i 's/^Server=.*/Server=/g' /etc/zabbix/zabbix_agentd.conf
```
Resultado:
```
Server=
```
&nbsp;
* Configurar o endereço do zabbix server
```
sed -i 's/^ServerActive=.*/ServerActive=device.external.net.br/g' /etc/zabbix/zabbix_agentd.conf
```
Resultado:
```
ServerActive=device.external.net.br
```
&nbsp;
* Configurar o hostname do agente zabbix
```
sed -i 's/^Hostname=.*/Hostname=server.01.svr/g' /etc/zabbix/zabbix_agentd.conf
```
Resultado:
```
Hostname=server.01.svr
```
&nbsp;
* Habilitar que o zabbix execute scripts personalizados
```
sed -i "s|# UnsafeUserParameters=0|UnsafeUserParameters=1|g" /etc/zabbix/zabbix_agentd.conf
```
Resultado:
```
UnsafeUserParameters=1
```
&nbsp;

* Adicionar um parâmetro de usuário para coletar a latência de ping
```
cat <<'EOF' >> /etc/zabbix/zabbix_agentd.conf
####### USER-DEFINED MONITORED PARAMETERS #######
#UserParameter=
UserParameter=user.ping.latency.[*], /opt/zabbix/ping-latency.sh $1
EOF
```
Resultado:
```
####### USER-DEFINED MONITORED PARAMETERS #######
#UserParameter=
UserParameter=user.ping.latency.[*], /opt/zabbix/ping-latency.sh $1
```
&nbsp;
* Criar o script para coletar a latência de ping
```
mkdir -p /opt/zabbix
cat <<'EOF' > /opt/zabbix/ping-latency.sh
#!/bin/bash
#Ping Destination
/usr/bin/fping $1 -C 1 2>&1 |  awk -F' ' '{print $6}'
EOF
```
Resultado:
```
#!/bin/bash
#Ping Destination
/usr/bin/fping $1 -C 1 2>&1 |  awk -F' ' '{print $6}'
```
&nbsp;
* Atribuir permissões ao script
```
chown -R root:zabbix /opt/zabbix/ping-latency.sh
chmod 775 /opt/zabbix/ping-latency.sh
```
&nbsp;
* Reiniciar o serviço do agente do Zabbix
```
systemctl restart zabbix-agent.service
```
&nbsp;
* Instalar o pacote fping
```
apt install fping -y
```

Passo 2: Configurar o Item na Interface Web do Zabbix
-----------------------------------------------------

O próximo passo é configurar o item de ping reverso na interface web do Zabbix. Para fazer isso, siga os seguintes passos:

1.  Faça login na interface web do Zabbix.
2.  Navegue até a seção "Configuration" e selecione "Hosts".
3.  Selecione o host para o qual você deseja adicionar o item de ping reverso.
4.  Na guia "Items", clique no botão "Create item".
5.  Na página "Create item", insira as seguintes informações:
    *   Name: um nome descritivo para o item, por exemplo, "Ping reverso".
    *   Type: "Zabbix agent".
    *   Key: "user.ping.latency\[hostname\]", substituindo "hostname" pelo nome ou endereço IP do host que você deseja testar.
    *   Type of information: "Numeric (float)".
    *   Units: "ms".
    *   Update interval: o intervalo de tempo em que o item será atualizado. Por exemplo, você pode definir 30 segundos.
    *   Applications: adicione um aplicativo relevante para o item, como "Network".
6.  Clique em "Add" para salvar o item.
7.  Verifique se o item está coletando dados corretamente navegando


Contribuição
------------

Contribuições são bem-vindas! Se você encontrar algum problema ou tiver sugestões para melhorar a aplicação, sinta-se à vontade para abrir uma issue ou enviar um pull request.

Se você gostou do meu trabalho e quer me agradecer, você pode me pagar um café :)

<a href="https://www.paypal.com/donate/?hosted_button_id=SFR785YEYHC4E" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 40px !important;width: 150px !important;" ></a>


Licença
-------

Este projeto está licenciado sob a Licença MIT. Consulte o arquivo LICENSE para obter mais informações.
