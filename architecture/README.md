
# Arquitetura AWS

De forma geral, a arquitetura desta implementação consiste no uso de 2 serviços principais: Amazon SageMaker e o Amazon S3. Como mostrado no diagrama abaixo, a base de dados crua foi salva em um _bucket_ S3 no diretório **/data/raw/** para ser consumida via um jupyter notebook criado no SageMaker. A primeira etapa do _notebook_ é responsável pela transformação e normalização dos dados originais no formato esperado pela API do SageMaker para treino do modelo, em que a estrutura consiste em um arquivo CSV sem cabeçalho que a primeira coluna seja a alvo. Após a aplicação do _pipeline_ de transformação os arquivos são separados em treino e teste e seu _upload_ é feito em subpastas diferentes no mesmo _bucket_ (**/data/train/** e **/data/test/**, respectivamente). A etapa de treino consiste na criação de um container do tipo Linear Learning que será responsável pela definição e treino do modelo. Este container terá acesso direto ao __/data/__ do _bucket_ utilizado. Os artefatos gerados pelo treino do modelo serão salvos no mesmo _bucket_, mas no diretório __/model/output/__. Como ultima etapa do _notebook_, uma inferência de um único _sample_ é feita via um _endpoint_ criado após o treino e a deleção do _endpoint_ por final (Clean Up).

![Diagrama da arquitetura utilizada para treino e deploy do modelo.](https://i.imgur.com/KPjyayc.jpeg)


## Amazon S3

Apenas um único *bucket* foi criado para esta esteira, sendo divido entre **model** e **data**, como visto na imagem abaixo. A pasta data está divida em 3 subpastas, sendo **raw** para os arquivos originais, e **train** e **test** para a base tratada e divida.
![Divisão do bucket usado](https://i.imgur.com/8Z3VybF.jpg)
Devido a sensibilidade dos dados armazenados, optou-se pela criação do *bucket* com encriptação automática dos dados armazenados, como mostrado na figura seguinte.
![Bucket encriptado](https://i.imgur.com/h0VsfZh.jpg)


## Amazon SageMaker

O notebook criado para execução do *pipeline* consiste nas configurações padrões sugeridas pela Amazon, apenas alterando a instância para a ml.m5.xlarge (4 vCPUs e 16GiB de RAM custando $0,23 por hora), como mostrado a seguir.
![Sagemaker configuração](https://i.imgur.com/S0rfoW6.jpg)

O container utilizado para treino do modelo foi o ml.m4.xlarge, que é pertencente ao nível gratuito do Amazon SageMaker e utilizamos a imagem **linear-learner** na versão **1**.

![Linear Estimator máquina.](https://i.imgur.com/FmwDEdB.jpg)

Nas configurações descritas acima, o modelo temorou cerca de 6 minutos para completar a função **fit()** e subir o modelo no S3.

![Tempo de treino.](https://i.imgur.com/jvHFmCw.jpg)

O endpoint criado utiliza uma a máquina ml.c4.xlarge, que também faz parte do nível gratuito do SageMaker, e permite acessos externos.
![Endpoint criado.](https://i.imgur.com/65MJnUQ.jpg)

Ao final, o *endpoint* usado para inferência foi deletado através do trecho abaixo como boa prática, mesmo não tendo custos adicionais para este caso (nível gratuito).
![cleanup endpoint](https://i.imgur.com/V7hoJIa.jpg)
