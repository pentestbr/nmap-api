NMAP API
==============

A NMAP API é uma API para disponibilização de resultados encontrados pelo NMAP e ferramentas semelhantes (que compartilhem o mesmo formato de saída).


Instalação
----------

Para usar a NMAP API você precisa instalar os seguintes modulos Perl:

* [MongoDB](https://metacpan.org/pod/MongoDB) -- Usado para integração com o banco de dados
* [Mojolicious::Lite](http://mojolicio.us/perldoc/Mojolicious/Lite) -- Usado como framework web
* [Mojo::JSON](http://mojolicio.us/perldoc/Mojo/JSON) -- Usado para ler JSON como um hash
* [Mojo::Log](http://mojolicio.us/perldoc/Mojo/Log) -- Usado para o envio de eventos localmente
* [Readonly](https://metacpan.org/pod/Readonly) -- Usado para gerar as constantes
* [Hash::Merge](https://metacpan.org/pod/Hash::Merge) -- Usado para mesclar resultados (útil quando o resultado vem de lugares diferentes)
* [Net::Syslog](https://metacpan.org/pod/Net::Syslog) -- Usado para o envio de eventos via syslog
* [Nmap::Parser](https://metacpan.org/pod/Nmap::Parser) -- Usado para fazer o parser dos arquivos XML do NMAP
* [Scalar::Util::Numeric](https://metacpan.org/pod/Scalar::Util::Numeric) -- Usado para verificar se determinada variável é inteira, através do uso da função `isint`
* [Data::Validate::IP](https://metacpan.org/pod/Data::Validate::IP) -- Usado para verificar se uma variável é um endereço IPv4 ou IPv6 válido, através das funções `is_ipv4` e `is_ipv6`
* [XML::Twig](https://metacpan.org/pod/XML::Twig) -- Usado para validar se o XML enviado para importação é válido
* [NetAddr::IP](https://metacpan.org/pod/NetAddr::IP) -- Usado para gerar todos os endereços IP dentro de uma rede

É recomendável que você faça uso do [carton](https://metacpan.org/pod/Carton) para não modificar o Perl instalado em seu sistema e fazer a gestão das dependencias de forma mais fácil. Outras alternativas são usar o [perlbrew](http://perlbrew.pl/), [plenv](https://github.com/tokuhirom/plenv) ou [local::lib](https://metacpan.org/pod/local::lib).

Usando o carton, para instalar as dependências, basta executar na pasta da aplicação:

	carton install

Após instalar é necessário configurar as variáveis de ambiente da aplicação. Elas são necessárias tanto para iniciar a aplicação quanto para pará-la.

Configuração
------------

A configuração da API é toda feita por variáveis de ambiente. Um exemplo de configuração pode ser visto a seguir:

	export NMAP_API_MONGO_HOST="localhost"
	export NMAP_API_MONGO_PORT="27017"
	export NMAP_API_DATABASE="nmap"
	export NMAP_API_HOST_COLLECTION="hosts"
	export NMAP_API_SCAN_COLLECTION="scans"
	export NMAP_API_LOG="LOCAL"
	export NMAP_API_URL='http://*:8080'
	export NMAP_API_WORKERS=16
	export NMAP_API_USER=nobody
	export NMAP_API_GROUP=nobody

Nesse exemplo, colocamos os eventos para serem gerados localmente, logo deverá ser criada no diretório da aplicação uma pasta chamada `log`.

No exemplo a seguir, configuramos para o envio de eventos para um coletor remoto (em 192.168.0.32):

	export NMAP_API_MONGO_HOST="localhost"
	export NMAP_API_MONGO_PORT="27017"
	export NMAP_API_DATABASE="nmap"
	export NMAP_API_HOST_COLLECTION="hosts"
	export NMAP_API_SCAN_COLLECTION="scans"
	export NMAP_API_LOG="NET"
	export NMAP_API_SYSLOG_PORT="514"
	export NMAP_API_SYSLOG_HOST="192.168.0.32"
	export NMAP_API_URL='http://*:8080'
	export NMAP_API_WORKERS=16
	export NMAP_API_USER=nobody
	export NMAP_API_GROUP=nobody

Nesse exemplo, os eventos serão enviados via Syslog para o host 192.168.0.32, na porta 514.


Iniciando
---------

Se você estiver usando o Carton, para iniciar a aplicação, você pode executar:

	carton exec hypnotoad nmap-api.pl

E para parar você pode executar:

	carton exec hypnotoad -s nmap-api.pl

Caso queira modificar a aplicação e deseja que ela reinicie a cada mudança no código, use:

	carton exec morbo nmap-api.pl


Uso
---

A NMAP API é uma API que tenta seguir os padrões REST. Assim, ela disponibiliza de forma fácil o obtenção e envio de dados.


### Contagem de hosts (GET /api/1.0/hosts/count)

	http://localhost:3000/api/1.0/hosts/count

Produz uma resposta:

	{"total":2038,"result":"success"}

Onde `total` é a quantidade de hosts na base.


### Listagem de hosts (GET /api/1.0/hosts)

	http://localhost:3000/api/1.0/hosts

Produz uma resposta:

	[
    	"192.168.200.10",
    	"192.168.200.1",
    	"192.168.200.21"
	]


### Contagem de scans (GET /api/1.0/scans/count)

	http://localhost:3000/api/1.0/scans/count

Produz uma resposta:

	{"total":1939,"result":"success"}

Onde `total` é a quantidade de scans na base.


### Listagem de scans (GET /api/1.0/scans)

	http://localhost:3000/api/1.0/scans

Produz uma resposta:

	[
		"1424012777",
		"1424012789"
	]

Onde cada item é o timestamp do scan. No caso de scans com o mesmo timestamp, somente um é mostrado.


### Obtendo item (GET /api/1.0/hosts/`<IP>`)

	http://localhost:3000/api/1.0/hosts/192.168.24.1

Produz uma resposta com os dados do host pedido:

	{
	    "distance": null,
	    "status": "up",
	    "mac_addr": "00:18:74:15:F2:80",
	    "data": [{
	        "proto": "tcp",
	        "rpc": null,
	        "version": null,
	        "fingerprint": null,
	        "service": "telnet",
	        "port": "23",
	        "additional_info": {
	            "banner": {
	                "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	            }
	        },
	        "product": "Cisco router"
	    }],
	    "hostnames": ["rt.example.net"],
	    "os": null,
	    "scans": [1390747279],
	    "os_family": null,
	    "mac_vendor": "Cisco Systems",
	    "uptime": null,
	    "addr": "192.168.24.1"
	}


### Obtendo item de determinado scan (GET /api/1.0/hosts/`<IP>`?scan=`<scan>`)

	http://localhost:3000/api/1.0/hosts/192.168.24.1?scan=1390747279

Produz uma resposta com os dados do host pedido:

	{
	    "distance": null,
	    "status": "up",
	    "mac_addr": "00:18:74:15:F2:80",
	    "data": [{
	        "proto": "tcp",
	        "rpc": null,
	        "version": null,
	        "fingerprint": null,
	        "service": "telnet",
	        "port": "23",
	        "additional_info": {
	            "banner": {
	                "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	            }
	        },
	        "product": "Cisco router"
	    }],
	    "hostnames": ["rt.example.net"],
	    "os": null,
	    "scans": [1390747279],
	    "os_family": null,
	    "mac_vendor": "Cisco Systems",
	    "uptime": null,
	    "addr": "192.168.24.1"
	}


### Obtendo itens de uma rede (GET /api/1.0/net/`<network>`/`<mask>`)

	http://localhost:3000/api/1.0/net/192.168.24.0/30

Produz uma resposta com os dados da rede pedida:

	{
	    "192.168.24.1": {
	        "distance": null,
	        "status": "up",
	        "mac_addr": "00:18:74:15:F2:80",
	        "data": [{
	            "proto": "tcp",
	            "rpc": null,
	            "version": null,
	            "fingerprint": null,
	            "service": "telnet",
	            "port": "23",
	            "additional_info": {
	                "banner": {
	                    "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	                }
	            },
	            "product": "Cisco router"
	        }],
	        "hostnames": ["rt.example.net"],
	        "os": null,
	        "scans": [1390747279],
	        "os_family": null,
	        "mac_vendor": "Cisco Systems",
	        "uptime": null,
	        "addr": "192.168.24.1"
	    }
	}


### Obtendo itens de uma rede com determinada porta aberta (GET /api/1.0/net/`<network>`/`<netmask>`?port=`<port>`)

	http://localhost:3000/api/1.0/net/192.168.24.0/30?port=23

Produz uma resposta com os dados da rede pedida segundo o critério estabelecido:

	{
	    "192.168.1.185": {
	        "distance": null,
	        "status": "up",
	        "mac_addr": null,
	        "data": [{
	            "proto": "tcp",
	            "rpc": null,
	            "version": null,
	            "fingerprint": null,
	            "service": "telnet",
	            "port": "23",
	            "additional_info": {
	                "banner": {
	                    "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	                }
	            },
	            "product": "Cisco router"
	        }, {
	            "proto": "tcp",
	            "rpc": null,
	            "version": null,
	            "fingerprint": null,
	            "service": null,
	            "port": "4786",
	            "additional_info": {},
	            "product": null
	        }],
	        "hostnames": [],
	        "os": null,
	        "scans": [1390744214],
	        "os_family": null,
	        "mac_vendor": null,
	        "uptime": null,
	        "addr": "192.168.1.185"
	    },
	    "192.168.1.129": {
	        "distance": null,
	        "status": "up",
	        "mac_addr": null,
	        "data": [{
	            "proto": "tcp",
	            "rpc": null,
	            "version": null,
	            "fingerprint": null,
	            "service": "telnet",
	            "port": "23",
	            "additional_info": {
	                "banner": {
	                    "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	                }
	            },
	            "product": "Cisco router"
	        }],
	        "hostnames": ["rt1.example.net"],
	        "os": null,
	        "scans": [1390744210],
	        "os_family": null,
	        "mac_vendor": null,
	        "uptime": null,
	        "addr": "192.168.1.129"
	    },
	    "192.168.1.131": {
	        "distance": null,
	        "status": "up",
	        "mac_addr": null,
	        "data": [{
	            "proto": "tcp",
	            "rpc": null,
	            "version": null,
	            "fingerprint": null,
	            "service": "telnet",
	            "port": "23",
	            "additional_info": {
	                "banner": {
	                    "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	                }
	            },
	            "product": "Cisco router"
	        }],
	        "hostnames": ["rt2.example.net"],
	        "os": null,
	        "scans": [1390744210],
	        "os_family": null,
	        "mac_vendor": null,
	        "uptime": null,
	        "addr": "192.168.1.131"
	    }
	}

Embora o uso dessa funcionalidade seja para buscar em redes, é possível usá-la para verificar se um host possui determinado serviço tratando tal como uma rede de um host. Veja os exemplos a seguir:

	http://localhost:3000/api/1.0/net/192.168.24.1/32?port=23

Produz a resposta:

	{
	    "distance": null,
	    "status": "up",
	    "mac_addr": "00:18:74:15:F2:80",
	    "data": [{
	        "proto": "tcp",
	        "rpc": null,
	        "version": null,
	        "fingerprint": null,
	        "service": "telnet",
	        "port": "23",
	        "additional_info": {
	            "banner": {
	                "output": "\\xFF\\xFB\\x01\\xFF\\xFB\\x03\\xFF\\xFD\\x18\\xFF\\xFD\\x1F\\x0D\\x0A\\x0D..."
	            }
	        },
	        "product": "Cisco router"
	    }],
	    "hostnames": ["rt.example.net"],
	    "os": null,
	    "scans": [1390747279],
	    "os_family": null,
	    "mac_vendor": "Cisco Systems",
	    "uptime": null,
	    "addr": "192.168.24.1"
	}

O que indica que o host ( /32 ) possui determinada porta aberta.

Caso o host não possua a porta aberta, a respota seria:

	{}

Já que nenhum host da rede possui tal critério.


### Obtendo itens de uma rede com determinado serviço (independente da porta que o serviço está executando) (GET /api/1.0/net/`<network>`/`<mask>`?service=`<service>`)

	http://localhost:3000/api/1.0/net/192.168.24.0/30?service=telnet

A resposta é semelhante a anterior.


### Obtendo itens de uma rede com determinado serviço em uma porta específica (GET /api/1.0/net/`<network>`/`<mask>`?service=`<service>`&port=`<port>`)

	http://localhost:3000/api/1.0/net/192.168.24.0/30?service=telnet&port=23

A resposta é semelhante a anterior.


### Inserindo item (PUT /api/1.0/scans)

	curl --upload-file nmap_result.xml 'http://localhost:3000/api/1.0/scans'

Produz uma resposta conforme a seguir em caso de sucesso:

	{"result":"success"}

Caso ocorra erros, a API responde:

	{"code":"112","result":"error","message":"Servico invalido (caracteres nao permitidos)!"}

Ou, caso o erro não seja rastreável:

	{"result":"error"}


Problemas Encontrados
--------------------

Resultados do NMAP Script Engine que contenham . (pontos) apresentam erros na importação.

Scripts mapeados com problemas:

* ssl-enum-chiphers - TLSv1.0


Licenciamento
-------------

Esse software é livre e deve ser distribuido sobre a Perl Artistic Licence.


Autor
-----

Copyrigth [Manoel Domingues Junior](http://github.com/mdjunior) <manoel at ufrj dot br>

