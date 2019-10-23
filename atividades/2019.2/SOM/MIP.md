# Mapeamento de �ndice de Aprova��o
## Introdu��o
Este trabalho consiste em analisar um banco de dados pelo m�todo SOM de machine learning.
O problema em quest�o consiste em analisar as primeiras semanas de um semestre em uma mat�ria e tentar determinar as situa��es de cada discente, dividindo eles em perfis de caracteriza��o.
Cada perfil possui uma parcecla de aprovado e de reprovado. O c�digo ser� respons�vel por dividir cada usu�rio dentro dessas parcelas, podendo assim entender melhor o comportamente de cada aluno ao longo da disciplina.
A base de dados consiste em descrever as notas de cada atividade semanal submetida de cada aluno. Contendo um total de 21 semanas e, consequentemente, 21 atividades.
Foi utilizado, tamb�m, uma �rvore de decis�es por fins de compara��o, para analisar o comportamento l�gico comparado ao c�digo SOM realizado.

### Contribuidores

- Samuel Amico Fidelis, aluno da Universidade Federal do Rio Grande do Norte do curso de bacharelado em engenharia mecatr�nica. Para eventuais d�vidas, entrar em contato pelos
meios abaixo:
* [Github](https://github.com/samuelamico/MachineLearning)
* [Site](https://samuelamico.github.io/)

- Leonardo Queiroz, aluno de bacharelado em engenharia mec�nica na Universidade Federal do Rio Grande do Norte, contato em:
* [Github](https://github.com/leocqueiroz)

- Gabriel Varela, aluno da Universidade Federal do Rio Grande do Norte do curso de bacharelado em engenharia mecatr�nica. Contato em:
* [Github](https://github.com/gabrielvrl)
* [Site](https://gabrielvrl.github.io/)

## Metodologia

### SOM
O SOM, self-organizing map, � uma rede neural artificial sem supervisionamento que � respons�vel por criar um mapa dimensionado que descretiza a base de dados, al�m de organiz�-las de forma adequada
A base SOM possui uma fase de treino, a qual ser� respons�vel por treinar a rede neural a partir dos valores de input escritos na base de dados. Ap�s o processo de treino, ele passar� por uma fase de mapeamento, a qual sera respons�vel por classificar de forma autom�tica a base de dados em neur�nios, organizando esses neur�nios de forma autom�tica.

#### Experimento e C�digos

O c�digo para a resolu��o desse problema pode ser encontrado no Github de Gabriel: [Github](https://github.com/gabrielvrl/Machine-Learning-ECT/blob/master/SOM_U2.ipynb).

Primeiro passo � importar as bibliotecas a serem utilizadas e o banco de dados, vamos utilizar o pandas pois � uma biblioteca que fornece ferramentas de an�lise de dados e estruturas de alta performance e f�ceis de usar. E tamb�m o numpy, para permitir trabalhar com arrays, vetores e matrizes.
Assim, utilizamos o "read_csv" da biblioteca pandas para fazer a leitura dos dados do reposit�rio do Github.

```py
import pandas as pd
import numpy as np
dataSet =  pd.read_csv("https://raw.githubusercontent.com/ect-info/ml/master/dados/lop_submissao_semana.csv",index_col=False )

dataSet['situacao'] = pd.Categorical(dataSet['situacao'])
dataSet['situacao'] = dataSet['situacao'].cat.codes

dataSet.head()
```

Seguido da instala��o da MiniSOM, a MiniSOM � uma implementa��o minimalista dos Self-Organizing Map(SOM)

```py
!pip install minisom
```

Dessa forma, podemos realizar o treinamento da nossa rede:
Treinamos com todas as vari�veis semanais, como "semana 1, semana 2, semana 3,..., semana 21".

```py
X_train = dataSet.iloc[:,2:23].values 
target_train = dataSet.iloc[:,25].values

[row, col] = X_train.shape
print (row," ",col)

print(X_train[1,:])
```

Definimos o tamanho em X e Y da rede (7x7) e utilizando o minisom, passamos os par�metros (x,y,input_len, sigma e learning_rate), assim, fazemos o treinamento.

```py

# Training the SOM
tamanhoXdaRede = 7; 
tamanhoYdaRede = 7; 

quantidadeCaracteristicas = col
from minisom import MiniSom
som = MiniSom(x = tamanhoXdaRede, y = tamanhoYdaRede, input_len = quantidadeCaracteristicas, sigma = 1.0, learning_rate = 0.4)
som.pca_weights_init(X_train)

som.train_random(data = X_train, num_iteration = 40000)
```

Como no m�todo SOM n�o existe uma fase, de fato, de teste, apenas uma an�lise de dados, utilizamos o c�digo abaixo que ser� respons�vel por criar nossa matriz resultado. Essa matriz � o mapeamento posto em pr�tica. Ela ser� responsavel por identificar os perfis de cada usu�rio e divid�-los entre as categorias.

```py
# encontra o vencedor 
x = X_train[1,:]
pos = som.winner(x)


# matriz de zeros para contador de aprovados 
MContAp = np.zeros((tamanhoXdaRede,tamanhoYdaRede))
# matriz de zeros para o contador de reprovados 
MContT = np.zeros((tamanhoXdaRede,tamanhoYdaRede))
cont = 0; 
for x in X_train: 
  pos = som.winner(x)
  if (Y_train[cont] <= 1): #Aprovado 
    MContAp[pos] += 1
  MContT[pos] += 1
  cont= cont+1
```

Ap�s a cria��o da matriz � separado as porcentagens entre Aprovado e Reprovado para cada perfil, chegando no resultado dispon�vel na imagem abaixo:

![AprovadoReprovado](https://github.com/leocqueiroz/MachineLearning/blob/master/SOM/Imagens/AprovadoReproado.PNG)

Onde, na imagem, a cor azul representa Aprova��o e a laranja Reprova��o.

Ap�s essa divis�o, � poss�vel encontrar o mapeamento de pesos, a partir do c�digo abaixo:

```py
# Mostra todos os pesos 
cont = 1;
x = np.arange(quantidadeCaracteristicas)
for row in pesos:
  for elem in row:
    plt.subplot(tamanhoXdaRede,tamanhoYdaRede,cont)
    cont=cont+1
    plt.axis([-1, 6, 0, 10])
    plt.bar(x, elem)
    plt.plot([-1,6],[5,5],'r')
plt.show()
#plt.savefig("test_som.jpg", dpi=150)
```

O resultado dos pesos desse mapeamento � mostrado na seguinte imagem

![Pesos](https://github.com/leocqueiroz/MachineLearning/blob/master/SOM/Imagens/Pesos.PNG)

Sendo assim, ent�o, poss�vel observar os resultados do SOM, uma vez que o mesmo n�o possui de fato uma fase teste, e sim uma fase de an�lise do mapeamento gerado.

### DecisionTree

�rvores de decis�o s�o m�todos de aprendizado de m�quinas supervisionado n�o-param�tricos, muito utilizados em tarefas de classifica��o e regress�o. �rvores, de um modo geral em computa��o, s�o estruturas de dados formadas por um conjunto de elementos que armazenam informa��es chamadas n�s.
Em uma �rvore de decis�o, uma decis�o � tomada atrav�s do caminhamento a partir do n� raiz, maior n�vel hier�rquico (o ponto de partida) at� o n� folha ou terminal.
A �rvore de decis�o foi utilizada para comparar os resultados obtidos utilizando o modelo SOM anteriormente.

#### PowerBI

Foi utilizado o PowerBI para construir um pain�l para que se pudesser analisar e escolher os melhores dados de semanas para treinar o modelo de �rvore de decis�o. A m�trica utilizada levou em considera��o as primeiras semanas 
, pois � preciso descobrir os alunos que est�o em dificuldade logo para que se possa dar uma aten��o melhor neles, logo em seguida foi analisado no painel PowerBI que as 5 ou 6 primeiras semanas demonstraram
um comportamento bem diferente entre elas.

O painel do PowerBI est� disponivel pelo link: [PowerBI](https://github.com/samuelamico/MachineLearning/tree/master/DecisionTree)

#### Experimento e C�digos

Escolhido as semanas para serem treinadas atrav�s de fun��es da biblioteca sklearn:

```py
from sklearn.model_selection import train_test_split # Import train_test_split function

x = dataset_filter.loc[:,[dataset_filter['semana 1'],dataset_filter['semana 2'],dataset_filter['semana 3'],dataset_filter['semana 4'],dataset_filter['semana 5'],dataset_filter['semana 6']]]
y = dataset_filter.iloc[:,-1]
feature_cols = ['semana 1','semana 2','semana 3','semana 4','semana 5','semana 6']

X_train, X_test, y_train, y_test = train_test_split(x, y, test_size=0.3, random_state=1) 
```

Ent�o logo em seguida foi criado o objeto �rvore de decis�o e utilizando o dataset filtrada para as semanas mais importantes temos:

```py
LopTree = DecisionTreeClassifier(criterion="entropy", max_depth = 4)
LopTree # it shows the default parameters
LopTree.fit(X_train,y_train)
```

Foi obtida ent�o a seguinte acur�cia para o uso de �rvore de decis�o:

```py
from sklearn import metrics
print("Accuracy:",metrics.accuracy_score(y_test, predTree))

Accuracy: 0.4456140350877193
```

Portanto, a acur�cia n�o foi t�o satisfat�ria, por�m foi testado utilizando outras combina��es de semanas e percebeu que 
realmente as seis primeiras semanas eram as melhores possiveis escolhas tomando como base o resultado de acur�cia, como ja indicava no pain�l do PowerBI. Portanto mesmo que a acur�cia n�o tenha sido t�o satisfatoria utilizando o m�todo de �rvore de decis�o
vale ressaltar a import�ncia deste resultado de acur�cia, pois mostra que realmente as seis primeiras semanas s�o as mais importantes para se obter um melhor resultado de previs�o.

A imagem abaixo mostra a �rvore de decis�o:

![Arvore](https://github.com/samuelamico/MachineLearning/blob/master/DecisionTree/loptree.png)