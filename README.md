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