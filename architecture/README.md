# Arquitetura

## Introdução

Esse notebook é utilizado para fazer o deploy do modelo desenvolvido na disciplina de Aprendizado de Máquina do Prof.º Samuel, no curso de Ciência de Dados do IFSP Campinas. Após realizada as etapas de pré-processamento, separação das bases, treinamento do modelo, fine tunning e validação dos resultados, selecionamos o modelo vencedor para ser instanciado na nuvem, respeitando as boas práticas de modelo rodando em produção com capacidade de avaliar novas entradas de dados. 

O problema abordado é um classificador binário para predição de indíviduos com diabetes. Por ser diagnosticada através de exames laboratoriais, estima-se que 1 em cada 5 pessoas (nos Estados Unidos), tenham a doença mas não foram diagnosticadas. Nossa tentativa foi de encontrar um  modelo capaz de predizer de forma eficaz e que não dependesse de exames clínicos as chances do índividuo ter a doença, principalmente nos casos sem diagnóstico. 

Para isso, chegamos ao modelo vencedor: SVM com kernel RBF que foi melhor em detectar os casos positivos em uma dada população. Utilizamos portanto os recursos disponíveis na AWS para criar uma solução capaz de treinar, armazenar os resultados e utilizar a inteligência gerada para predição de novas entradas de dados. 

## Metodologia

Utilizamos para este trabalho os dados resumidos da pesquisa feita pelo CDC, que consiste em dados tabulares que foram coletados em entrevistas via telefone de individuos residentes nos Estados Unidos. 

É uma base de aproximadamente 22Mb que possui 256,680 registros. Além de contar com 22 colunas, contendo informações sobre o comportamento e indicadores socio-econômicos dos índividuos entrevistados. Os tipos de dados presentes na tabela podem ser descritos conforme abaixo: 

- ``Diabetes_binary`` - Our target. 0 = no diabetes; 1 = diabetes
- ``HigBP`` - High Blood Presure. 0 = no HighBP; 1 = HighBP
- ``HighChol`` - High Cholesterol. 0 = no HighChol; 1 = HighChol
- ``CholCheck`` - Cholesterol Check in the last 5 years. 0 = no cholesterol check; 1 = yes cholesterol check
- ``BMI`` - Body Mass Index. [More info](https://en.wikipedia.org/wiki/Body_mass_index)
- ``Smoker`` - Indicates if the person have smoked at least 100 cigarrets in your entire life. 0 = no; 1 = yes
- ``Stroke`` - Have ever had a stroke. 0 = no; 1 = yes
- ``HeartDiseaseorAttack`` - Have ever had a heart disease or attack. Coronary hear disease or myocardial infraction. 0 = no; 1 = yes
- ``PhysActivity`` - Have praticed physical activity in past 30 days (not including job). 0 = no; 1 = yes
- ``Fruits`` - Have consumed fruit 1 or more times per day. 0 = no; 1 = yes
- ``Veggies`` - Have consumed vegetables 1 or more times per day. 0 = no; 1 = yes
- ``HvyAlcoholConsump`` - If men having more than 14 drinks per week or if women having more than 7 drinks per week. 0 = no; 1 = yes
- ``AnyHealthcare`` - Have any kind of health care coverage. 0 = no; 1 = yes
- ``NoDocbcCost`` - Was there a time on the past 12 years when you nedded to see a doctor but could not because of cost. 0 = no; 1 = yes
- ``GenHlth`` - Would you say that in general your health in a scale between 0 to 5 is. 1 = excellent; 2 = very good; 3 = good; 4 = fair; 5 = poor; 7 = don’t know/not Sure; 9 = refused.
- ``MentHlth`` - Now thinking about your mental health, which includes stress, depression, and problems with emotions, for how many days during the past 30 days was your mental health not good? 1 - 30 = number of days
- ``PhysHlth`` - Now thinking about your physical health, which includes physical illness and injury, for how many days during the past 30 days was your physical. 1 - 30 = number of days
- ``DiffWalk`` - Do you have serious difficulty walking or climbing stairs? 0 = no; 1 = yes
- ``Sex`` - 0 = female; 1 = male
- ``Age`` - 1 = 18-24; 2 = 25-29; 3 = 30-34; 4 = 35-39; 5 = 40-44; 6 = 45-49; 7 = 50-54; 8 = 55-59; 9 = 60-64; 10 = 65-69; 11 = 70-74; 12 = 75-79; 13 = 80 or older
- ``Education`` - 1 = Never attended school; 2 = Elementary school; 3 = High school incomplete; 4 = High school graduate; 5 = Some college or technical school; 6 = College graduate; 9 = refused to answer
- ``Income`` - Is your annual household income from all sources.<br/>
1 = Less than $10,000;  2 = Less than $15,000 ($10,000 to less than $15,000);<br/>
3 = Less than $20,000 ($15,000 to less than $20,000);  4 = Less than $25,000 ($20,000 to less than $25,000)<br/>
5 = Less than $35,000 ($25,000 to less than $35,000);  6 = Less than $50,000 ($35,000 to less than $50,000)<br/>
7 = Less than $75,000 ($50,000 to less than $75,000);  8 = $75,000 or more <br/>

Todas as colunas são do tipo float64.

## Recursos AWS

De forma geral, a arquitetura desta implementação consiste no uso de 2 serviços principais: 
* Amazon SageMaker 
* Amazon S3

Como mostrado no diagrama abaixo, a base de dados crua foi salva em um _bucket_ S3 no diretório **/data/raw/** para ser consumida via um jupyter notebook criado no SageMaker. 

Nossa solução é baseadas no deploy em produção de um pipeline que executa algumas tarefas:

* Transformação e normalização dos dados
* Divisão dos dados entre treino e teste
* Criar um estimador linear
* Salvar os resultados obtidos pelo estimador em um bucket S3
* Criar um endpoint para recuperar os dados de treinamento e rodar novas predições
* Por último executar uma tarefa de limpeza dos recursos utilizados para evitar custos desnecessários

![Diagrama da arquitetura utilizada para treino e deploy do modelo.](https://i.imgur.com/KPjyayc.jpeg)

### Descrição detalhada do pipeline

1. A primeira etapa do _notebook_ é responsável pela transformação e normalização dos dados originais no formato esperado pela API do SageMaker para treino do modelo, em que a estrutura consiste em um arquivo CSV sem cabeçalho que a primeira coluna seja a alvo. 

2. Após a aplicação do _pipeline_ de transformação os arquivos são separados em treino e teste e seu _upload_ é feito em subpastas diferentes no mesmo _bucket_ (**/data/train/** e **/data/test/**, respectivamente).

3. A etapa de treino consiste na criação de um container do tipo Linear Learning que será responsável pela definição e treino do modelo. Este container terá acesso direto ao __/data/__ do _bucket_ utilizado. 

4. Os artefatos gerados pelo treino do modelo serão salvos no mesmo _bucket_, mas no diretório __/model/output/__. 

5. Como ultima etapa do _notebook_, uma inferência de um único _sample_ é feita via um _endpoint_ criado após o treino e a deleção do _endpoint_ por final (Clean Up).

## Detalhamento Recursos utilizados


### Amazon S3

Apenas um único *bucket* foi criado para esta esteira, sendo divido entre **model** e **data**, como visto na imagem abaixo. A pasta data está divida em 3 subpastas:
* **raw** - Armazenamento dos arquivos originais sem alteração
* **train** - Armazenamento do resultado do treinamento
* **test** - Armazenamento dos dados não vistos utilizados para validação do modelo

![Divisão do bucket usado](https://i.imgur.com/8Z3VybF.jpg)


Devido a sensibilidade dos dados armazenados, optou-se pela criação do *bucket* com encriptação automática dos dados armazenados, como mostrado na figura:<br>

![Bucket encriptado](https://i.imgur.com/h0VsfZh.jpg)</br>


## Amazon SageMaker

O notebook criado para execução do *pipeline* usa as configurações padrão sugeridas pela Amazon, alteramos apenas a instância para a ml.m5.xlarge:

* Machine: *ml.m5.xlarge*
  * 4 vCPUs
  * 16GiB RAM

Esta máquina tem um custo estimado de U$0,23/hora de uso;

![Sagemaker configuração](https://i.imgur.com/S0rfoW6.jpg)

Para o treinamento a instância utilizada foi a ml.m4.xlarge, que está disponível no nível gratuito do SageMaker. Utilizamos a imagem **linear-learner** na versão **1**, como descrito na imagem:

![Linear Estimator máquina.](https://i.imgur.com/FmwDEdB.jpg)

Nas configurações descritas acima, o modelo temorou cerca de 6 minutos para completar a função **fit()** e subir o modelo no S3.

![Tempo de treino.](https://i.imgur.com/jvHFmCw.jpg)

O endpoint criado utiliza uma a máquina ml.c4.xlarge, que também faz parte do nível gratuito do SageMaker, e permite acessos externos.
![Endpoint criado.](https://i.imgur.com/65MJnUQ.jpg)

Ao final, o *endpoint* usado para inferência foi deletado através do trecho abaixo como boa prática, mesmo não tendo custos adicionais para este caso (nível gratuito).<br>

![cleanup endpoint](https://i.imgur.com/V7hoJIa.jpg)


## Referências

1. [Notebook Exemplo - PySpark Mnist](https://github.com/BiancaPedrosa/amazon-sagemaker-examples/blob/master/sagemaker-spark/pyspark_mnist/pyspark_mnist_kmeans.ipynb)
2. [Amazon Linear Learner](https://aws.amazon.com/pt/blogs/machine-learning/train-faster-more-flexible-models-with-amazon-sagemaker-linear-learner/)
