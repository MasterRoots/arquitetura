# Prova de Arquitetura de Soluções

### Arquitetura e Designer (10 pontos)
Você, sendo um dos arquitetos da principal plataforma de e-commerce da Companhia, é chamado para uma reunião técnica para falar sobre algumas metas que as lojas precisam ter para atingir o target de 300k usuários simultâneos, considerando primeiro acesso, pesquisas, detalhamento de pedido e finalização de compra. Para tanto, o Diretor da área decide que não devemos mais evoluir a plataforma atual e pede para que o Arquiteto desenhe uma nova solução de Arquitetura contendo os seguintes requisitos:
- Seja escalável;
- Seja Segura;
- Seja Flexível para atender o negócio, principalmente ao front;
- Rode em ambiente de Cloud Computing;
- Tenha resiliência nas conectividades e integrações;
- Seja Evolutiva na parte tecnológica, e para manutenção e Testes;
- Tenha Baixa latência para a camada de Banco de Dados;

#### Proposta
Desenhe uma plano arquitetural/ Arquitetura de referencia envolvendo todas as camadas da aplicação, forma de integração entre as camadas e forma de integração para outros sistemas. Descreva ainda quais tecnologias devem ser colocadas em cada uma destas camadas e porque deve ser utilizadas cada uma delas tecnologias.
Entrega

Os desenhos da solução, podem ser entregues via Google Docs, Office 365, Git, ArchimateTool, Gliffy etc. So não esqueçam de autorizar o acesso.

Para atender aos requisitos propostos, desenhei uma arquitetura de referência separadas em dois momentos.

Uma arquitetura simplificada de infraestrutura e outra para aplicação.

## Arquitetura de Referência Infraestrutura

![](/img/ARQ1-1.png)

No modelo proposto, estamos utilizando como cloud, a infra da Microsoft Azure. Independente do uso dessa ou mesmo de outra como por exemplo, a  AWS, o uso de containers gerenciados pelo Kubernetes é uma estratégia que pode ser usada em ambos os casos e nos permite vários ganhos, como por exemplo:

- Escalabilidade e economia de recursos, pois conseguimos otimizar a utilização dos mesmos baseados no uso de cada APP e de sua sasonalidade.

- Também conseguimos isolar de forma segura as aplicações e gerencia-las

- Comunicação facilitada entre aplicações, etc

	O uso da cloud também nos auxilia nas questões de resiliência e tolerância a falhas, disponibilizando nossa infra em regiões diferentes em casos de crise, e claro ser ajustado a uma região mais próxima da nossa localidade diminuindo Latência.

Nosso desenho também informa antes da entrada da nossa infra, ainda nos utilizamos de uma CDN( no caso a Akamai) aonde conseguimos entregar caches de nosso APP, melhorando nossa entrega ao consumidor final.


A segurança nessa camada também é realizada usando de WAF, aonde conseguimos usar a técnica de firewall e também bloquear acessos nocivos a nossa infra, bloqueado Ip's por exemplo que por acaso venham estar honerando nossos recursos com o uso de um Brute Force.

Dentro do nosso ambiente cloud, também podemos trabalhar por aplicação e seu uso de um banco de dados. A latência do uso pode ser minimizado com a escolha da região que o banco estará, se é melhor usar uma maquina virtualizada para hospeda-lo ou se vale a pena utilizar um banco em SAAS aonde passo toda a gestão de tolerãncia, vazão elasticidade para a cloud. Sempre atentando para que isso pode gerar uma depedência com a cloud escolhida. 

A parte de CI e automação de deploys fica a cargo do famoso Jenkins, aonde podemos criar pipelines para deploys automatizados na estrura de cloud, ainda permitindo a execução de testes de integração, unitários, regressivos e etc.

Outro ponto interessante nessa criação é o modelo de gerenciamento, extração e visualização de logs da aplicação se utilizando das aplicações fluentd, kafka, graylog e elastic, respectivamente gerenciador de logs, mensageria que enviados os dados para serem idexados pelo eslatic e organizados para pesquisas no graylog.


Finalizando essa primeira estrutura, ainda temos a possibilidade de extrair as informações coletadas em todos nossos banco filtrando com Apache Spark criando um Data Lake. Esse processo tira a necessidade de aplicações tipicas de BI, honerarem a rede em busca de informações, queries lentas blocando bases e recursos e informações mais apuraradas para serem consumidas.


## Arquitetura de Aplicação

![](/img/ARQ1-2.png)

oi elaborado, baseado em meu conhecimento adquirido na netshoe, um modelo simplificado das aplicações 	que compoem nosso e-commerce

A stack é basicamente criada a partir do padrão de microserviços. Como linguagem de programação temos aplicações em Java, utilizando spring-boot como framework e node.js com Vue.js aonde criamos o front da loja.

Na parte de segurança e comunicação, as aplicações de autenticação via oAuth2, ecompartilhando um token para validar suas transações.

O negócio de um e-commerce é bem dinâmico, e cada etapa do funel de compra é atendido por um serviço diferente.

Como loja, temos que permitir flexibilidade tanto para o time de marketing, quanto para a criação de outras lojas se utilizando da mesma infraestrutura.

Isso é alcançado, aonde podemos utilizar o conceito de templastes por loja. Isso define a caracteristica de cada loja, como identidade visual, troca de banners e etc.

A pesquisa é um ponto critico nesse processo, caso demore muito podemos perder nosso cliente. Por isso, não podemos simples fazer consultas direto em um banco comum. Mesmo com o uso correto de indices, utilização de estratégias de shards e etc, o melhor que podemos fazer para otimizar a pesquisa é criar arquivos indexavéis que facilitam a pesquisa, por isso nesse ponto nos utilizamos de elastic search.

Outro processo que é bem concorrido nesse aspecto, é a aplicação de preços. Esse cálculo pode variar, mas os processo das lojas mudam em uma velocidade maior. Por isso nesse ponto utilizamos a tecnologia de chave e valor do Redis, como estrutura de cache.

Para todo esse processo, acabamos fazendo muitas integrações, tanto interno como externamente. Por isso não podemos deixar nosso site vulnerável a quedas e lentidão. Nesse fluxo, cálculamos frete, consultamos produtos, aplicamos promoções e muito mais, podemos inclusive falar com empresas terceiras para fazer uma pagamento, uma análise de risco e etc. A estratégia de fallbacks e requisões assíncronas são fundamentais.

Sempre temos que ter alternativas para o desastre e ter rotas de fuga para opções default da loja e concluírmos a compra.

Para o caso de falha, fallbacks automáticos e manuais, para processos de negócios aonde não existe a necessidade de entregar uma informação para o usuário, processos assíncronos com o uso de mensageria.
No nosso caso RabbitMQ e Kafka.

