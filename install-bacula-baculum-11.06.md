## Instalar Bacula e Baculum 11.06 no Rocky Linux 9.0 ##

**Todos os comandos serão com usuário root**

Fazer update e upgrade do sistema:
```
yum update -y && yum upgrade -y
```
Instalar postgres server com o comando:
```
dnf install postgresql-server
```
Iniciar o setup postgresql:
```
postgresql-setup --initdb
```
Iniciar o serviço do postgresql:
```
systemctl start postgresql
```
Habilitar o postgres no boot do sistema: 
```
systemctl enable postgresql
```

## Instalar pacotes necessários para instalar e compilar o Bacula:
```
yum -y install nano wget tar gcc-c++ readline-devel zlib-devel lzo-devel libacl-devel mt-st mtx postfix openssl-devel postgresql-devel
```
Fazer um diretório temporário para o bacula usando os comandos
```
cd ..
mkdir baculatemp
```

Entrar no diretório:
```
cd baculatemp
```

## Baixar o bacula 11.06 com o comando:
```
wget -O bacula-11.0.6.tar.gz https://sourceforge.net/projects/bacula/files/bacula/11.0.6/bacula-11.0.6.tar.gz
```
Descompactar o arquivo com o comando:
```
tar -zxvf nome_arquivo_tar.gz
```
Mudar para o diretório onde foi descompactado:
```
cd bacula-11.0.6
```
No diretório onde foi descompactado o mesmo, fazer o comando para compilar o bacula: Atenção para os parâmetros: **--with-job-email=seu-e-mail-desejado --with-hostname=hostname-ou-IP-desejado:**
```
./configure --with-readline=/usr/include/readline --disable-conio --bindir=/usr/bin --sysconfdir=/opt/bacula/etc --sbindir=/usr/sbin --with-scriptdir=/opt/bacula/scripts --with-working-dir=/opt/bacula/working --with-logdir=/var/log–enable-smartalloc --with-postgresql --with-archivedir=/mnt/backup --with-job-email=e-mail-desejado --with-hostname=IP-desejado
```

O resultado deve ser:
```
Configuration on Wed Sep 21 08:22:53 EDT 2022:

   Host:                      x86_64-pc-linux-gnu -- redhat (Blue
   Bacula version:            Bacula 11.0.6 (10 March 2022)
   Source code location:      .
   Install binaries:          /usr/sbin
   Install libraries:         /usr/lib64
   Install config files:      /opt/bacula/etc
   Scripts directory:         /opt/bacula/scripts
   Archive directory:         /mnt/backup
   Working directory:         /opt/bacula/working
   PID directory:             /var/run
   Subsys directory:          /var/lock/subsys
   Man directory:             /usr/share/man
   Data directory:            /usr/share
   Plugin directory:          /usr/lib64
   C Compiler:                gcc 11.2.1
   C++ Compiler:              /usr/bin/g++ 11.2.1
   Compiler flags:             -g -O2 -Wall -x c++ -fno-strict-aliasing -fno-exceptions -fno-rtti
   Linker flags:               
   Libraries:                 -lpthread 
   Statically Linked Tools:   no
   Statically Linked FD:      no
   Statically Linked SD:      no
   Statically Linked DIR:     no
   Statically Linked CONS:    no
   Database backends:         PostgreSQL
   Database port:              
   Database name:             bacula
   Database user:             bacula
   Database SSL options:      

   Job Output Email:          e-mail-desejado
   Traceback Email:           root@localhost
   SMTP Host Address:         localhost

   Director Port:             9101
   File daemon Port:          9102
   Storage daemon Port:       9103

   Director User:             
   Director Group:            
   Storage Daemon User:       
   Storage DaemonGroup:       
   File Daemon User:          
   File Daemon Group:         

   Large file support:        yes
   Bacula conio support:      no -lreadline -lhistory -ltinfo
   readline support:          yes 
   TCP Wrappers support:      no 
   TLS support:               yes
   Encryption support:        yes
   ZLIB support:              yes
   LZO support:               yes
   S3 support:                no
   enable-smartalloc:         yes
   enable-lockmgr:            no
   bat support:               no
   client-only:               no
   build-dird:                yes
   build-stored:              yes
   Plugin support:            yes
   AFS support:               no
   ACL support:               yes
   XATTR support:             yes
   GPFS support:              no 
   systemd support:           no 
   Batch insert enabled:      PostgreSQL

   Plugins:
- Docker:                  no
```

E então fazer o comando:
```
make -j 8 && make install
```
Criar um usuário bacula com a senha bacula no postgres. No terminal mudar para o usuário postgres:
```
su - postgres
```
Executar o comando:
```
psql
```

Criar o usuário bacula com o password bacula com o comando:
```
CREATE USER bacula WITH PASSWORD 'bacula';
```
Para sair:
```
\q
exit
```
No Terminal mudar as permissões do diretório executando o comando:
```
chmod 775 /opt/bacula/
```
Mudar para o diretório /opt/bacula/scripts
```
cd /opt/bacula/scripts
```

Mudar o proprietário para postgres dos seguintes arquivos com o comando:
```
chown postgres create_postgresql_database && chown postgres make_postgresql_tables  
chown postgres grant_postgresql_privileges && chown postgres drop_postgresql_database  
chown postgres update_postgresql_tables
```
Trocar o usuário para postgres:
```
su - postgres
```
Mudar para o diretório:
```
cd /opt/bacula/scripts/
```

```
Criar a base de dados do postgresql,
Fazer as tabelas,
Garantir privilégios,
Sair com os comandos
```
Então, com os comandos:
```
./create_postgresql_database 
./make_postgresql_tables
./grant_postgresql_privileges  
exit
```
Editar o arquivo **/var/lib/pgsql/data/postgresql.conf**:
```
nano /var/lib/pgsql/data/postgresql.conf
Antes: # listen_addresses = ‘localhost’

Para:
listen_addresses = '*'

De: #port = 5432
Para:
port = 5432

Salvar e sair do arquivo
```

Editar o arquivo **/var/lib/pgsql/data/pg_hba.conf**:

## Antes:

```
nano /var/lib/pgsql/data/pg_hba.conf

# “local” is for Unix domain socket connections only
local  all all  peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
```

## Para:

```
# "local" is for Unix domain socket connections only
local   all             all                                   **md5**
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
```

Editar o arquivo bacula-dir.conf e inserir as credenciais para se conectar a porta 5432:
```
nano /opt/bacula/etc/bacula-dir.conf
```

Procurar pela linha Catalog e insira as linhas como abaixo:
```
Catalog {  
Name = MyCatalog  
dbdriver = "postgresql"; dbaddress = 127.0.0.1; dbport = 5432  
dbname = "bacula"; dbuser = "bacula"; dbpassword = "bacula"
}

Salvar o arquivo
```
## Fazer um arquivo Unit do serviço Bacula

Mudar para o diretório:
```
cd /etc/systemd/system
```
Criar o arquivo:
```
nano bacula.service
```
Copie e cole com o seguinte conteúdo para seu arquivo:
```
[Unit]  
Description=Bacula backup service  
After=syslog.target network.target

[Service]  
Type=forking  
ExecStart=/opt/bacula/scripts/bacula start  
ExecReload=/opt/bacula/scripts/bacula reload  
ExecStop=/opt/bacula/scripts/bacula stop

[Install]  
WantedBy=multi-user.target
Salvar o arquivo
```

Recarregar o daemon do systemd:
```
systemctl daemon-reload
```
Iniciar o Serviço bacula:
```
systemctl start bacula.service
```
Verificar o Status do serviço bacula:
```
systemctl status bacula.service
```
Habilitar o serviço bacula para iniciar com o sistema:
```
systemctl enable bacula.service
```
Reiniciar o servidor:
```
reboot
```

Testando o Bacula

Lembrando que os serviços do **Postgresql** e **Bacula** devem estar ativos. Verificar com os comandos:
```
systemctl status bacula.service
systemctl status postgresq
```

