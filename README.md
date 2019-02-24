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

## Solução

Para atender aos requisitos propostos, desenhei uma arquitetura de referência separadas em dois momentos.

Uma arquitetura simplificada de infraestrutura e outra para aplicação.

## Arquitetura de Referência Infraestrutura

![](/img/ARQ1-1.png)

No modelo proposto, estamos utilizando como cloud, a infra da Microsoft Azure. Independente do uso dessa ou mesmo de outra como por exemplo, a  AWS, o uso de containers gerenciados pelo Kubernetes é uma estratégia que pode ser usada em ambos os casos e nos permite vários ganhos, como por exemplo:

- Escalabilidade e economia de recursos, pois conseguimos otimizar a utilização dos mesmos baseados no uso de cada APP e de sua sasonalidade.

- Também conseguimos isolar de forma segura as aplicações e gerencia-las

- Comunicação facilitada entre aplicações, etc

- O uso da cloud também nos auxilia nas questões de resiliência e tolerância a falhas, disponibilizando nossa infra em regiões diferentes em casos de crise, e claro ser ajustado a uma região mais próxima da nossa localidade diminuindo Latência.

Nosso desenho também informa antes da entrada da nossa infra, ainda nos utilizamos de uma CDN( no caso a Akamai) aonde conseguimos entregar caches de nosso APP, melhorando nossa entrega ao consumidor final.


A segurança nessa camada também é realizada usando de WAF, aonde conseguimos usar a técnica de firewall e também bloquear acessos nocivos a nossa infra, bloqueando Ip's por exemplo que por acaso venham estar honerando nossos recursos com o uso de um Brute Force.

Dentro do nosso ambiente cloud, também podemos trabalhar por aplicação e seu uso de um banco de dados. A latência do uso pode ser minimizado com a escolha da região que o banco estará, se é melhor usar uma maquina virtualizada para hospeda-lo ou se vale a pena utilizar um banco em SAAS aonde passo toda a gestão de tolerãncia, vazão elasticidade para a cloud. Sempre atentando para que isso pode gerar uma depedência com a cloud escolhida. 

A parte de CI e automação de deploys fica a cargo do famoso Jenkins, aonde podemos criar pipelines para deploys automatizados na estrura de cloud, ainda permitindo a execução de testes de integração, unitários, regressivos e etc.

Outro ponto interessante nessa criação é o modelo de gerenciamento, extração e visualização de logs da aplicação se utilizando das aplicações fluentd, kafka, graylog e elastic, respectivamente gerenciador de logs, mensageria que enviados os dados para serem idexados pelo eslatic e organizados para pesquisas no graylog.


Finalizando essa primeira estrutura, ainda temos a possibilidade de extrair as informações coletadas em todos nossos banco filtrando com Apache Spark criando um Data Lake. Esse processo tira a necessidade de aplicações tipicas de BI, honerarem a rede em busca de informações, queries lentas blocando bases e recursos e informações mais apuraradas para serem consumidas.


## Arquitetura de Aplicação

![](/img/ARQ1-2.png)

Foi elaborado, baseado em meu conhecimento adquirido na netshoes, um modelo simplificado das aplicações 	que compoem nosso e-commerce

A stack é basicamente criada a partir do padrão de microserviços. Como linguagem de programação temos aplicações em Java, utilizando spring-boot como framework e node.js com Vue.js aonde criamos o front da loja.

Na parte de segurança e comunicação, as aplicações de autenticação via oAuth2, e compartilhando um token para validar suas transações.

O negócio de um e-commerce é bem dinâmico, e cada etapa do funel de compra é atendido por um serviço diferente.

Como loja, temos que permitir flexibilidade tanto para o time de marketing, quanto para a criação de outras lojas se utilizando da mesma infraestrutura.

Isso é alcançado, aonde podemos utilizar o conceito de templastes por loja. Isso define a caracteristica de cada loja, como identidade visual, troca de banners e etc.

A pesquisa é um ponto critico nesse processo, caso demore muito podemos perder nosso cliente. Por isso, não podemos simplesmente fazer consultas direto em um banco comum. Mesmo com o uso correto de indices, utilização de estratégias de shards e etc, o melhor que podemos fazer para otimizar a pesquisa é criar arquivos indexavéis que facilitam a pesquisa, por isso nesse ponto nos utilizamos de elastic search.

Outro processo que é bem concorrido nesse aspecto, é a aplicação de preços. Esse cálculo pode variar, mas os processo das lojas mudam em uma velocidade maior. Por isso nesse ponto utilizamos a tecnologia de chave e valor do Redis, como estrutura de cache.

Para todo esse processo, acabamos fazendo muitas integrações, tanto interno como externamente. Por isso não podemos deixar nosso site vulnerável a quedas e lentidão. Nesse fluxo, cálculamos frete, consultamos produtos, aplicamos promoções e muito mais, podemos inclusive falar com empresas terceiras para fazer uma pagamento, uma análise de risco e etc. A estratégia de fallbacks e requisões assíncronas são fundamentais.

Sempre temos que ter alternativas para o desastre e ter rotas de fuga para opções default da loja e concluírmos a compra.

Para o caso de falha, fallbacks automáticos e manuais, para processos de negócios aonde não existe a necessidade de entregar uma informação para o usuário, processos assíncronos com o uso de mensageria.
No nosso caso RabbitMQ e Kafka.


### 2) Limitação Backoffice (10 pontos)

Mais uma vez, você é chamado para uma reunião de tecnologia para discutir uma grande problema que está impactando a finalização de vários pedidos. Este problema está relacionado ao Back office onde é feito todo o processamento do pedido. Atualmente o pedido é iniciado na loja e finalizado no back office, mas, a versão atual do back office é um produto de empresa XPTO onde existe um limite de processamento de 5000 pedidos hora. Porém a meta de venda da Loja é de 40k e esta integração está aferindo diretamente esta meta pois o back office não processo em tempo hábil a cada integração. Você, como arquiteto deve resolver esse problema.

#### Proposta
Elabore um arquitetura de integração que resolva este problema. Nesta arquitetura, descreva as tecnologias a serem utilizadas e porque esta mudança resolve os problemas. Por fim, é necessário que seja utilizado um designer Pattern de Integração conhecido. Não esqueça de descreve-lo.

#### Entrega
Os desenhos da solução, podem ser entregues via Google Docs, Office 365, Git, ArchimateTool, Gliffy etc. So não esqueçam de autorizar o acesso.

## Solução


![](/img/ARQ2.png)


####Padrão proposto : 

####Competing Consumers

A caracteristica desse padrão é aonde meu app SENDER, coloca mensagens concorrentes em um sistema de mensageria, no caso o Kafka.

O Kafka por sua vez, cria tópicos que podem ser consumidos por múltiplos CONSUMERS.

Sendo assim, conseguimos aumentar o processamento por horas do sistema consumidor

Referência do Pattern

[Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html)


###3) Um pouco além do MicroServiço (5 pontos)

Como a ascensão de modularização e segregação de aplicações por contexto, explique com suas palavras:
- que é Micro Serviço?
- Porque devemos utilizar?
- Quais São os Prós e Contras.

Por fim, imagine que você, como arquiteto direcionou o time a construção de um micro serviço de CEP, por exemplo, e o mesmo é altamente acessado como, você como arquiteto, escalaria este micro service? Como resolveria a questão do "S.E.P" (Single entry point) ?

O que espera-se como resposta - Dicas e direcionamentos:
- Descreva detalhadamente todas as questões se forma organizada


## Solução

O que é Micro Serviço?

É um serviço que atende a um dominio especifico, que tenha baixo acoplamento e alta coesão. Diferentemente do modelo antigo monolitico que agregava todas as funcionalidades de um software em um "bloco único de código"

• Porque devemos utilizar?

Existem alguns benefícios da utilização desse modelo:

	- Funcionalidades mais consistentes e de dominio exclusivo,
	- Facilidade na integração,
	- Deploys mais consisos e independentes
	- Módulos reaproveitáveis no ambito geral do sistema.
	- Testes focados no contexto do negócio,
	- Divisão especifica de responsabilidades em squads
	- Diminui pontos únicos de falha no sistema
	- Facilita a escalabilidade



### Quais São os Prós e Contras.

Além dos pontos positivos declarados acima, devemos ter em mente que a utilização de microserviços trazem grandes beneficios para o sistema, mas ao mesmo tempo gera muito mais complexibilidade.

Os contras, ou talvez, pontos de atenção para a urilização de microserviços podem ser resumidos em:

- Monitoria mais complexa, aonde temos que verificar a performance e a saúde de várias aplicações
- Utilização de infraestrutura mais para agregra logs de várias aplicações
- Testes de integração mais complexos e processos de deploys que podem ficar mais inflados e demorarem mais para serem concluídos, ou seja, demanda mais atenção no que e no que não testar
- Mais atenção as estratégias de deploys, como canário, green/blue etc.


### Por fim, imagine que você, como arquiteto direcionou o time a construção de um micro serviço de CEP, por exemplo, e o mesmo é altamente acessado como, você como arquiteto, escalaria este micro service? Como resolveria a questão do "S.E.P" (Single entry point) ?


Como serviço independente em termos de contexto, a aplicação poderia ser escalada na horizontal. Como os dados referentes ao cep, não tem atualização constante, colocaria os dados em cache utilizando redis.

Os dados em geral estaria em um MongoDB, e esses dados do Redis, seriam "esquentados" por um scheduler, aonde seriam atualizados via aplicação de webservices dos correios, 2 vezes por semana.

Para termos um SEP, caracteristica de um api gateway, deveria ser desenolvido uma camada de aplicação que recebe essas requisões e fazem os roteamentos para as instâncias dos serviço de CEP.

Na frente disso tudo, temos que ter um Load Balancer, com inteligência de saber como distribuir o tráfego.
