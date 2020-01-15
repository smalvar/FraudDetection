# Detectando Fraude


**Situação:**

Recebo um conjunto de dados de uma empresa que administra um site de comércio eletrônico. Ele contém informações sobre as primeiras transações de muitos usuários diferentes, bem como se um determinado usuário passou a realizar uma atividade fraudulenta ou não.

**Objetivos:**

- Determinar o país de todos os usuários com base em seus endereços IP;
- Construir um modelo que prevê a probabilidade de que a primeira transação de um novo usuário seja fraudulenta. Explique como diferentes suposições sobre o custo de falsos positivos versus falsos negativos impactam o modelo.
- Explique ao seu chefe (que não é técnico em tudo!) como o modelo está fazendo previsões e por que eles deveriam confiar nele. Faça isso do ponto de vista do usuário, ou seja, descreva que tipo de usuários tem maior probabilidade de ser classificado em risco, quais são suas características, por que isso é razoável? Seu chefe tem medo de que você possa recusar os clientes pagantes válidos por acidente com essa ferramenta.
- Se o seu modelo estiver agora ativo e fazendo previsões em tempo real, descreva como ele deve ser usado a partir de uma perspectiva de produto, ou seja, que tipo de experiência de usuário você recomendaria que fosse criado com base na saída do modelo?

**Resultados:**

As características mais importantes na detecção de fraudes foram determinadas como:
- A velocidade na qual eles se movimentaram por todo o funil de vendas (da inscrição à compra)
- Se ou não vários IDs de usuário foram gerados a partir de um único dispositivo ou endereço IP
 
Acontece que nem todos os endereços IP no conjunto de dados podem ser rastreados para um país de origem. Não está claro se a lista de países e os limites de endereços IP estão incompletos, alguns endereços IP foram falsificados ou ocorreu um erro ao registrar o endereço IP. Independentemente, o país de origem não desempenha um papel significativo na determinação de fraudadores, por isso não importa realmente.
 
Eu construí dois modelos:

- Floresta de isolamento: um alogoritmo supervisionado baseado na análise de quantas divisões em recursos são necessárias para isolar uma determinada amostra.
- Uma máquina de vetores de suporte de classe: um algoritmo não supervisionado baseado no aprendizado de um limite de decisão envolvendo dados não anômalos e, em seguida, tentando identificar anomalias determinando sua distância a partir desse limite.
    
É importante identificar corretamente o comportamento fraudulento quando ele surge (queremos uma alta precisão) e não sinalizar os usuários normais como fraudulentos, alienando, assim, nossa base de clientes (queremos um recall alto). Para otimizar a precisão e a recordação simultaneamente, decidi otimizar a pontuação F1, a média harmônica de precisão e recuperação, em vez de precisão. Usando esse método de otimização, descobri que a floresta de isolamento tem um desempenho melhor do que a classe SVM. Embora o SVM tenha pontuações de precisão, recordação e F1 comparáveis à floresta de isolamento, ele não é tão flexível quanto a floresta de isolamento ao lidar com dados multimodais não gaussianos.

**Explicação não técnica:**

Embora os dois modelos desenvolvidos abordem o problema de maneira diferente, ambos estão tentando determinar qual é o comportamento "normal" do usuário em termos de quanto um usuário "normal" gasta, com que velocidade ele se move através do funil de vendas, sua idade, etc. são todos representados por números e esses números tenderão a ser valores um pouco semelhantes para usuários "normais". Para usuários fraudulentos, alguns desses números podem ser bem diferentes e é esse desvio do comportamento "normal" que esses modelos estão tentando descobrir. Os modelos aproveitam esses números e técnicas estatísticas avançadas para também caracterizar, com números, o quão diferente é um comportamento fraudulento do usuário de um usuário "normal". Usando essas técnicas, descobri que os usuários com maior probabilidade de serem classificados como um risco por esses modelos são aqueles que:

- 1) ter vários IDs de usuário associados a um único dispositivo ou endereço IP
- 2) passar de inscrição para compra em períodos extremamente curtos de tempo.

Para o ponto i) um usuário pode esquecer seu nome de usuário / senha e, portanto, se inscrever novamente, gerando um novo ID de usuário para o mesmo dispositivo ou endereço IP. No entanto, nesse caso, é altamente provável que apenas um ID de usuário seja usado por vez. Se vários IDs de usuário estiverem em uso simultaneamente, isso pode indicar que alguém configurou um botnet para interagir com o site. Além disso, muitas das atividades fraudulentas no conjunto de dados foram associadas a dispositivos ou endereços IP que utilizam até 10 IDs de usuário. Isso certamente não é algo que um usuário "normal" jamais faria.

Para o ponto ii) um período de tempo típico que potencialmente resultaria em uma bandeira vermelha de um desses algoritmos é de 1 a 2 segundos. Um usuário normal leva alguns minutos para simplesmente navegar no site, mesmo que já tenha tomado a decisão de comprar. Esse tempo é estendido se o usuário tiver que tomar alguma decisão sobre sua compra. Se todo o funil de vendas estiver sendo percorrido literalmente em 1 a 2 segundos, isso é novamente um sinal provável de que, na verdade, é um bot de algum tipo interagindo com o site.

**Recomendações:**

Eu recomendo implementar o modelo de floresta de isolamento em produção. Eu também tenho duas recomendações específicas para a equipe de experiência do usuário e uma para a equipe de segurança.

Certifique-se de exigir o nome completo e endereço de e-mail de um usuário para se inscrever. Quando um usuário esquecer seu nome de usuário / senha, dê a ele um caminho fácil de seguir para ** redefinir ** sua senha e exigir seu nome completo e endereço de e-mail para fazer isso. Dessa forma, o ID do usuário original do usuário pode ser encontrado no banco de dados e excluído antes de fornecer um novo ID de usuário. Isso minimiza as chances de um usuário normal ter vários IDs de usuário e reduz as chances de confundi-los com um fraudador.
Depois que um usuário se inscrever, apresente imediatamente ofertas / descontos em potencial, se possível. Isso pode ser anunciado como um simples agradecimento por você se inscrever, mas, para usuários normais, aumentará as chances de compra e, ao mesmo tempo, dará a eles algo a considerar, diminuindo um pouco a velocidade deles, dificultando confundi-los com eles. um bot que provavelmente ignoraria todas as ofertas de descontos.
Finalmente, também recomendo enfaticamente a segurança durante as primeiras três semanas do ano, pois parece haver um pico de atividade fraudulenta durante esse período.

