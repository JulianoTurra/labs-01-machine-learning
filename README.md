
# Aprendizado de Máquina Automatizado no Azure Machine Learning

Neste exemplo, iremos utilizar o recurso de aprendizado de máquina automatizado no Azure Machine Learning para treinar e avaliar um modelo de aprendizado.

## Criando uma workspace do Azure Machine Learning

Para usar o Azure Machine Learning, iremos criar um workspace utilizando Azure Machine Learning. Em seguida, iremos utilizar o Estúdio do Azure Machine Learning para trabalhar com os recursos em nossa workspace.

1. Entre no portal do Azure em https://portal.azure.com usando suas credenciais da Microsoft.

1. Selecione **+Criar um recurso**, pesquise Machine Learning e crie um recurso do Azure Machine Learning com as seguintes configurações:
    - **Assinatura**: *sua assinatura do Azure*.
    - **Grupo de recursos**: *crie ou selecione um grupo de recursos*.
    - **Nome**: *insira um nome exclusivo para o seu workspace*.
    - **Região**: *selecione a região geográfica mais próxima*.
    - **Conta de armazenamento**: *anote a nova conta de armazenamento padrão que será criada para o workspace*.
    - **Cofre de chaves**: *anote o novo cofre de chaves padrão que será criado para o workspace*.
    - **Application Insights**: *anote o novo recurso Application Insights padrão que será criado para o workspace*.
    - **Registro de contêiner**: *nenhum (será criado um automaticamente quando você implantar um modelo em um contêiner pela primeira vez)*.

1. Selecione **Examinar + criar**, então selecione **criar**. Aguarde até que o workspace seja criado (isso pode demorar alguns minutos) e acesse o recurso implantado.


1. Selecione **Iniciar o estúdio** (ou abra uma nova guia do navegador [https://ml.azure.com](https://ml.azure.com?azure-portal=true), e entre no Estúdio do Azure Machine Learning usando a conta Microsoft). Feche todas as mensagens exibidas.

1. No Estúdio do Azure Machine Learning, você verá uma workspace recém-criada. Caso contrário, selecione **Todos os workspaces** no menu à esquerda e selecione a workspace que você acabou de criar.

## Utilizando aprendizado de máquina automatizado para treinar um modelo

O aprendizado de máquina automatizado permite que seja feitos experimentos de vários algoritmos e parâmetros para treinar vários modelos e identificar o melhor para os nossos dados. Nesse caso, iremos usar um conjunto de dados de detalhes históricos de aluguel de bicicletas para treinar um modelo que prevê o número de aluguéis de bicicletas que deve ser esperado em um determinado dia, com base em características sazonais e meteorológicas.

> **Citação**: *Os dados usados neste exercício são derivados do [Capital Bikeshare](https://www.capitalbikeshare.com/system-data) e são usados de acordo com o [license agreement](https://www.capitalbikeshare.com/data-license-agreement)* de dados publicados.

1. Em [Azure Machine Learning studio](https://ml.azure.com?azure-portal=true), inicie a página **ML Automatizado** em **Criação**).

1. Crie um novo trabalho de ML automatizado com as seguintes configurações, usando **Avançar** conforme necessário para avançar dentro da interface do usuário:

    **Configurações básicas**:

    - **Nome do trabalho**: mslearn-bike-automl
    - **Nome do novo experimento**: `mslearn-bike-rental`
    - **Descrição**: machine learning automatizado para previsão de aluguel de bicicletas.
    - **Marcas**: *nenhuma*

   **Tipo de tarefa e dados**:

    - **Selecionar tipo de tarefa**: regressão
    - **Selecionar conjunto de dados**: crie um novo conjunto de dados com as seguintes configurações:
        - **Tipo de dados**:
            - **Nome**: `bike-rentals`
            - **Descrição**: `Historic bike rental data`
            - **Tipo**: Table (mltable)
        - **Fonte de dados**:
            - Selecionar  **Dos arquivos locais**
        - **Tipo de armazenamento de destino**:
            - **Tipo de armazenamento de dados**: Armazenamento do Blobs do Azure
            - **Nome**: workspaceblobstore
        - **Seleção de MLtable**:
            - **Carregar pasta**: *Baixe e descompacte a pasta contendo os dois arquivos necessários para carregar* `https://aka.ms/bike-rentals`

        Selecione **Criar**. Após a criação do conjunto de dados, selecione o conjunto de dados de **aluguel de bicicletas** para continuar e enviar o trabalho do ML Automatizado

    **Configurações da tarefa**:

    - **Tipo de tarefa**: regressão
    - **Conjunto de dados**: bike-rentals
    - **Coluna de destino**: aluguéis (inteiro)
    - **Definições de configuração adicionais**:
        - **Métrica primária**: NormalizedRootMeanSquaredError
        - **Explicar o melhor modelo**: *não selecionado*
        - **Habilitar empilhamento de conjunto**: *não selecionado*
        - **Usar todos os modelos com suporte**: Não selecionado. Você restringirá o trabalho para experimentar apenas alguns algoritmos específicos.
        - **Modelos permitidos**: *selecione apenas  **RandomForest** and **LightGBM** — O ideal seria tentar usar o máximo possível, mas cada modelo adicionado aumenta o tempo necessário para executar o trabalho.*
    - **Limites**: *expanda esta seção*
        - **Avaliações máximas**: `3`
        - **Máximo de avaliações simultâneas**: `3`
        - **Máximo de nós**: `3`
        - **Limite de pontuação da métrica**: `0.085` (*de modo que se um modelo atingir uma pontuação de métrica de raiz do erro quadrático médio normalizada de até 0,085, o trabalho será encerrado.*)
        - **Tempo limite do Experimento**: `15`
        - **Tempo limite de iteração**: `15`
        - **Habilitar encerramento antecipado**: *selecionado*
    - **Validação e teste**:
        - **Tipo de validação**: divisão de validação de treinamento
        - **Percentual de dados de validação**: 10
        - **Conjunto de dados de teste**: nenhum

    **Cálculo**:

    - **Selecionar tipo de cálculo**: sem servidor
    - **VTipo de máquina virtual**: CPU
    - **VCamada da máquina virtual**: dedicada
    - **Tamanho da máquina virtual**: Standard_DS3_V2\*
    - **Número de instâncias**: 1

    \* *Se sua assinatura restringir os tamanhos de VM disponíveis para você, escolha qualquer tamanho disponível.*

1. Envie o trabalho de treinamento. Ele será iniciado automaticamente.

1. Aguarde a conclusão do trabalho. Isso pode demorar um pouco.

## Examinar o melhor modelo

Quando o trabalho de aprendizado de máquina for concluído, vamos poder examinar o melhor modelo treinado
  
1. Selecione o texto em **Nome do algoritmo** do melhor modelo para exibir os respectivos detalhes.

1. Selecione a guia **Métricas ** e selecione os gráficos **residuals** e **predicted_true** se eles ainda não estiverem selecionados.

    Examine os gráficos que mostram o desempenho do modelo. O gráfico de **resíduos** mostra os *resíduos* (as diferenças entre valores previstos e reais) como um histograma. O gráfico **predicted_true** ompara os valores previstos com os valores verdadeiros.

## Implantando e testando o modelo

1. Na guia **Modelo** para obter o melhor modelo treinado pelo trabalho de aprendizado de máquina, selecione **Implantar** e use a opção **ponto de extremidade em tempo real** para implantar o modelo com as seguintes configurações:
    - **Máquina virtual:**: Standard_DS3_v2
    - **Contagem de instâncias**: 3
    - **Ponto de extremidade**: New
    - **Nome do ponto de extremidade**: *prever-aluguéis*
    - **Nome da implantação**: *Manter o padrão*
    - **Coleta de dados de inferência**: *Disabled*
    - **Empacotar modelo**: *Disabled*

1. Aguarde até o início da implantação – isso pode levar alguns segundos. O **Status de implantação** para o ponto de extremidade **predict-rentals** será indicado na parte principal da página como Em execução.
1. Aguarde até que o **Status de implantação** mude para Bem-sucedida. Isso pode levar de 5 a 10 minutos.

## Testar o serviço implantado

Agora iremos testar o serviço implantado.

1. Em Estúdio do Azure Machine Learning, no menu à esquerda, selecione **Pontos de Extremidade** e abra o ponto de extremidade em tempo real **predict-rentals**.

1. Na página do ponto de extremidade em tempo real de **previsão de aluguel** exiba a guia **Teste**.

1. No painel **Dados de entrada para testar o ponto de extremidade** substitua o modelo JSON pelos seguintes dados de entrada

    ```json
      {
     "input_data": {
       "columns": [
         "day",
         "mnth",
         "year",
         "season",
         "holiday",
         "weekday",
         "workingday",
         "weathersit",
         "temp",
         "atemp",
         "hum",
         "windspeed"
       ],
       "index": [0],
       "data": [[1,1,2022,2,0,1,1,2,0.3,0.3,0.3,0.3]]
     }
    }

    ```

1. Clique no botão **Testar**.

1. Revise os resultados do teste, que incluem um número previsto de aluguéis com base nos recursos de entrada – semelhante ao seguinte:

    ```JSON
    [
      345.58493704310524
    ]
    ```

    O painel de teste pegou os dados de entrada e utilizou o modelo que treinamos para retornar o número previsto de locações.

Utilizamos um conjunto de dados históricos de locação de bicicletas para treinar um modelo. O modelo prevê o número de locações de bicicletas esperadas em um determinado dia, com base em recursos sazonais e meteorológicos.



