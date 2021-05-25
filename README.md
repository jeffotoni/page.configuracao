# Page Saúde

### Sistema Operacional

Qualquer distribuição linux poderá ser utilizada para instalação e configuração do aplicativo.
O que usamos em nossos servidores foi o Ubuntu 16.04.7 LTS Server.

A forma de instalação e configuração da máquina podem utilizar conforme seu domínio e conhecimento para o sistema rodar não tem exisgências, porém pode ser interessante ter um server para o banco de dados e outro para nginx caso acharem interessante separar para melhorar o desempenho e manutenção.

O banco de dados sempre é interessante está em outro server mas isto é o que acharem mais interessante para o cenário de vocês.

Com relação a aplicação, neste cenário não encorajamos o uso do docker para subir a aplicação mas se acharem interessante e dominarem containers a aplicação irá funcionar sem nenhum problema porém precisa de algumas configurações para rodar em container.

### CRON 
Neste projeto existe a integração com um serviço utilizando docker para integração com serviço do client que utilizam Sql Server.
Criamos uma imagem e deixamos ela pública já com toda configuração que poderão utiliza-la: jeffotoni/mssql-php-msphpsql

O arquivo *cron/cron-script-get-sql-server.sh* é o que tem que configurar.

### ENVIO DE EMAILS

O sistema utiliza SES para envios de emails, ou nativo usando a função mail do PHP.
Para ambos precisaram configurar para que tudo funcione certinho.

Existe uma class que trata somente disto.
s3wfcore/classes/SgiMail.php método SgiSendMailS4.

### GERAR PDF

O sistema utiliza mpdf para gerar os PDFs então estes estão nativos, porém existe alguns PDFs utilizando um outro serviço para gera-los e para facilitar deixei o repo publico para que possam instalar e configura-lo para utiliza-lo em seu servidor.
serviço entrotra aqui: [gowkhtmlpdf](https://github.com/jeffotoni/gowkhtmltopdf)
Ele pode rodar em docker, e vc configura para que os relatórios possam utiliza-los.

### PHP

O seu server deverá rodar em LINUX, e a versão do PHP deverá ser PHP 7.0.15, porém sugiro instalar o ZendServer para rodar 100%.
Logo abaixo a versão do ZendServer que irão precisar para configurar.

Os módulos instalados do php são os defaults nada de especial foi utilizado abaixo deixei o link para visualizarem os pacotes.

[Visualizar modulos php](https://gist.github.com/jeffotoni/a260fcc9f712c4d4a2bf47e0c2e253f4)
[zendServer](https://s3wf.sfo2.digitaloceanspaces.com/ZendServer-9.0.2-RepositoryInstaller-linux.tar)

### Ngnix

O servidor web que utilizamos foi o nginx, e a configuração dele encontra-se no diretório docserver/nginx/page.producao.conf.
Neste diretório tem nosso nginx.conf só para terem uma ideia do que configuramos, podem usa-lo como exemplo para sua configuração se acharem necessário.

Vocês podem configurar o nginx da forma que acharem melhor, somente o arquivo page.producao.conf com os caminhos mapeados que são importantes são eles que vão direcionar o comportamento e funcionamento do sistema.

Um outro detalhe é que o nginx foi configurado para funcionar com uma nova extensão .s3 nos arquivos, irão precisar configurar isto.

Exemplo:
```bash
location ~ \.(s3|php)$ {
    #allow all;
    include fastcgi_params;
    fastcgi_pass unix:/usr/local/zend/tmp/php-fpm.sock;
}
```

### PostgreSQL
O banco de dados utilizado é o PostgreSQL, a versão é >= 12
O banco encontra-se no formato .dmp e para restaura-lo você terá que executar um pg_restore  e o formato .sql caso preferir para restaura-lo precisará executar o comando psql.

O usuário utilizado é o s3wf, o dmp e sql estão com estes usuarios, mas caso queiram alterar podem ficar a vontade, o exemplo abaixo é utilizando s3wf como usuario.

Você irá configurar o banco de dados conforme o esquema abaixo para que sua aplicação funcione corretamente.

```bash
$ createuser s3wf -U postgres
$ psql -d template1 -U postgres
> alter user s3wf CREATEROLE LOGIN;
> alter user s3wf CREATEDB;
> alter user s3wf CREATEROLE;
> alter user s3wf password 'your-senha';
> CREATE EXTENSION pgcrypto;
```

$ createdb page-producao -U s3wf -O s3wf -E UTF-8 -T template0 -h <your-server>
$ pg_restore -d page-producao page-producao.dmp -h <your-server> -U postgres


Lembrando que o postgres tem que ser configurado para os acessos basicamente em pg_hba e postgresql.conf.


### O sistema

A configuração do sistema para acesso ao banco de dados e seu funcionamento são feitos em somente 2 arquivos principais.

Diretório que encontra-se a configuração do banco de dados:

$ s3wfcore/config/.db/credentials
```bash
[config keys database]

domain	 = your-server
database = page-producao
password = your-passwrod
porta	 = 5432

```

Basta configura-lo conforme seu cenário.


Outro arquivo de configuração que temos que ficar atento é o local.conf.php.

$ s3wfcore/config/local.conf.php

```php
<?php

    // Pre definindo
    // variavel local ou nao
    // este file nao fica no repositorio git
    // 
    //
    define("LOCAL", false);

    #
    #define('AMBIENTE', 'TESTE');

    #
    #define('AMBIENTE', 'PRODUCAO');

    #
    define('AMBIENTE', 'PRODUCAO');

    // path fixado para producao
    define('PATH_SESSION', '/var/lib/php/session/page.producao');

    @session_save_path(PATH_SESSION);

```
O que muda aí é somente o path da SESSION se for usar session como file, caso usem Redis para session não há necessidade de configurar este path.

Toda a configuração é para produção, apesar da plataforma ter para diversos ambientes como desenvolvimento etc. esta configuração é para produção e a produção é configurada para rodar em https e o arquivo que determina isto é o configambiente.producao.conf.php que encontra-se em:

$ s3wfcore/config/configambiente.producao.conf.php na linha 49 [https], caso queiram rodar sem ssl basta alterar para [http]

Caso queiram configurar o timezone da aplicação conforme seu php.ini, está tudo no arquivo:
$ s3wfcore/config/configambiente.conf.php linha 50.

Na linha 47 temos ini_set('session.gc_maxlifetime', 86400), este nosso coleguinha é para configurar o tempo limite da session, fiquem a vontade em colocar o que for melhor para sua necessidade.
