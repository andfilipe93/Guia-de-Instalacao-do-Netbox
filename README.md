Guia de Instalação do NetBox no Ubuntu 22.04 LTS


Antes de começar, é altamente recomendável realizar um snapshot da máquina para facilitar possíveis rollbacks após atualizações.

Requisitos Mínimos:

Memória: 4GB (mínimo)
Python: >= 3.8
PostgreSQL: >= 10
Redis: >= 4.0
Atualizar a máquina:

# atualizar softwares da maquina
apt update

# instalar o postgres
apt install -y postgresql
# verificar a versão do postgres
psql -V

# criação da data base
sudo -u postgres psql
CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'J5brHrAXFLQSif0K';
ALTER DATABASE netbox OWNER TO netbox;
\connect netbox;
GRANT CREATE ON SCHEMA public TO netbox;

# instalação do Redis
apt install -y redis-server
# tem que retornar como "PONG" para ter certeza que serviço esta ok
redis-cli ping

# instalar o python
sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev
# rodar após a instalação do python com seus pacotes
pip3 install --upgrade pip

# Obs: instalar a versão mais atualizada conforme site da netbox, pois pode mudar a versão com o tempo
wget https://github.com/netbox-community/netbox/archive/refs/tags/vx.x.x.tar.gz
tar -xzf vx.x.x.tar.gz -C /opt
ln -s /opt/netbox
ls -l /opt | grep netbox

# criar usuário netbox
adduser --system -group netbox
chown --recursive netbox /opt/netbox/netbox/media/

# copiar pasta do arquivo principal
cd /opt/netbox/netbox/netbox/
cp configuration_example.py configuration.py

# informar o IP ou nome da VM (para testes pode inserir '*')
entrar dentro do arquivo configuration.py em "ALLOWED_HOSTS ="

# gerar chave nesse comando para colar detro do arquivo configuration.py
python3 ../generate_secret_key.py

# cria o ambiente virtual do python
echo napalm >> /opt/netbox/local_requirements.txt

# script para fazer o upgrade
cd /opt/netbox
./upgrade.sh

# ambiente virtual do python
source /opt/netbox/venv/bin/activate

# criar usuario para netbox
cd /opt/netbox/netbox
python3 manage.py createsuperuser

# verificar se esta funcional dentro do ambiente python (informar o ip da maquina seguido da porta do firewall)
python3 manage.py runserver 0.0.0.0:8000 --insecure
# verificar se a porta 8000 ou qualquer outra que informar esta habilitada no firewall

# comando "deactivate" sai do interface python

# arquivos de serviço do gunicorn
cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
cp -v /opt/netbox/contrib/*.service /etc/systemd/system/

systemctl daemon-reload
systemctl start netbox netbox-rq
systemctl enable netbox netbox-rq
# ativar serviços netbox
systemctl status netbox

# instalar servidor web
apt install -y nginx
cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
# entrar dentro do arquivo netbox.conf para informar ip da maquina e caso não tiver certificado comentar a 3 linhas do ssl
vi /etc/nginx/sites-available/netbox
# para subir as configurações alteradas
systemctl restart nginx


Com essas instruções, você terá o NetBox instalado e configurado no seu ambiente Ubuntu 22.04 LTS. Certifique-se de ajustar conforme necessário, especialmente versões e caminhos, e acompanhe possíveis atualizações do NetBox e suas dependências.
