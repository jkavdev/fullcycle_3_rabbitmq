# fullcycle_3_rabbitmq
Repositorio para o curso de RabbitMQ da Fullcycle

rabbitmq eh um message broker - sistema que permite a comunicacao entre o cliente e um vendedor, no caso duas partes interessadas em se comunicar
	no caso ele cuida do cliente que vai publicar a mensagem e o cliente que vai consumir a mensagem

ele implementa varios protocolos de comunicacao amqp, mqtt, stomp e http	
	o mais utilizado eh o amqp, baseado em tcp, sendo muito rapido o trafego de informacoes

ele guarda todas as mensagens em memoria	

ele eh padrao de mercado, muitas empresas o utilizam

na iteracao com o rabbitmq, temos o cliente que vai ser comunicar com o servidor/rabbitmq
essa comunicacao inicial eh feita com tcp, essa primeira conexao eh lenta, devido a isso
o rabbitmq criar apenas uma unica conexao e dentro dessa conexao
ele tem canais/channels que utiliza essa mesma conexao, isso chamado de multiplexing connections
toda vez que abrimos uma nova channel, estamos criando uma nova thread no servidor para essa channel criada

no funcionamento basico, temos um publicador de mensagens que ira publicar a mensagem para uma exchange
exchange fara o tratamente da mensagem e enviara para uma ou varias filas definidas
a mensagem postada na fila, os consumidores ficaram lendo essas filas para obter essas mensagens
o publisher nunca se comunica com a fila ou o consumidor diretamente, sempre publisher -> exchange -> queue - consumer

tipos de exchange, temos 
a direct - no qual ira mandar uma mensagem para uma fila especifica
a fanout - no qual ira mandar uma mensagem para varias filas no qual ela esta linkada/ligadas a essa exchange
a topic - consegue definir regras de acordo com a routing key/mensagem, aplicar e enviar essa mensagem com essas regras
a header - no header da mensagem eh definido para qual fila a mensagem sera enviada, eh a menos utilizada

o processo de ligar uma exchange a uma fila, se chama bind
uma exchange pode se comunicar com varias filas, e vice versa
a direct - eh definida uma routing key na mensagem e na fila, e quando essa mensagem cai na exchange, ela vai saber para qual fila mandar pela routing key ligada a cada fila
a topic - eh parecida com a direct, pois utiliza tambem routing keys, mas nesse caso podemos criar expressoes regulares informando que para tal padrao de routing key sera vinculado a uma fila
	fila a - routing key - a.* - mensagem - routing key - a.criar
	fila g - routing key - r.*.funcionar - mensagem - routing key - r.criar.funcionar

filas, por padrao as filas seguem FIFO, as primeiras mensagens que entram, sao as primeiras a sairem
nas filas temos algumas propriedades de configuracao
durable - indica se essa fila deve ser salva, mesmo depois do reinicar do broker
auto-delete - remover uma fila caso o consumer se desconecte da fila, pode ser que eh uma fila para um caso especifico, ai como o consumer ja finalizou sua tarefa, nao tem razao para essa fila existir mais
expiry - define o tempo que nao ha mensagens ou clientes consumindo, para ser removida caso nao tenha nenhuma atividade
message ttl - tempo de vida da mensagem	
overflow - definicoes quando a mensagem lota de mensagens
	drop head - remove a ultima mensagem, quando uma nova mensagem chega e ta lotada a fila
	reject public - rejeita a mensagem que chegando
exclusive - define que essa fila se torne exclusiva a um canal especifico	
max length ou bytes - quantidade de mensagens que a fila pode receber e o tamanho em bytes permitidos

curiosidades: nao falamos criar uma fila, e sim declarar um fila, porque nas apis de mensageria, vamos fazer um .declare(fila)

dead letter queues, quando temos mensagens que nao sao entregues por algum motivo, o rabbitmq, sabe dessas mensagens, mesmo que nao de sucesso
podemos configurar uma exchange e fila para essas mensagens que nao foram entregues, para serem processadas num cenario extraordinario

lazy queues, sao filas que armazenam as mensagens em disco, tem que ter um cenario bem especifico para poder utilizalas, pois o padrao sao filas que armazenam em memoria
tem um alta utilizacao de I/O, devido ao disco, utilizada normalmente quando o fluxo de mensagens eh tao grande que os consumidores nao conseguem ler todas as mensagens, ai para garantir a ordem das mensagens essas mensagens pode ser jogadas para essa fila

simulando o sistema de mensageria no rabbitmq
	https://tryrabbitmq.com/

confiabilidade	
o rabbitmq tem recursos que ajudam a termos confiabilidade na entrega de mensagens e saber se foi processada corretamente pelos consumidores
consumer aknowledgement
	temos tres 3 tipos de repostas nesse caso
	ack - indica que conseguiu processar com sucesso a mensagem
	reject - indica que nao conseguiu processar a mensagem, rejeitando a mensagem, mensagem por mensagem, fazendo que a mensagem retorna para a fila
	nack - eh a mesma coisa do reject, mas consegue rejeitar varias mensagens ao mesmo tempo

publisher confirm
	quando o publisher enviar a mensagem para a exchange, a exchange enviara uma resposta indicando que recebeu a mensagem
	para esse caso, eh criado um id unico para a mensagem, de responsabilidade do cliente em criar esse id
	temos 2 tipos
	ack - a exchange recebeu a mensagem
	nack - a exchange rejeita a mensagem por algum motivo

filas e mensagens duraveis/persistidas