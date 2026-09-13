---
lab:
    title: 'Analisar texto'
    description: "Use Azure Language no Foundry Tools para analisar texto."
    level: 300
    duration: 30
    islab: true
    status: 'released'
---

# Analisar texto

O **Azure Language no Foundry Tools** oferece suporte à análise de texto, incluindo language detection, entity recognition e PII redaction.

Por exemplo, suponha que uma agência de viagens queira processar avaliações de hotéis enviadas ao site da empresa. Usando o Azure Language, é possível determinar o idioma em que cada avaliação foi escrita, identificar named entities, como locais, pontos de referência ou pessoas mencionadas nas avaliações, e redigir qualquer personally identifiable information (PII) antes de publicá-las no site da empresa. Neste exercício, você usará o Azure Language Python SDK for text analytics para implementar um aplicativo simples de avaliação de hotéis.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de text analytics usando vários SDKs específicos de linguagem; incluindo:

O código usado neste exercício é baseado no Microsoft Foundry Tools SDK for Python. Você pode desenvolver soluções semelhantes usando os SDKs for Microsoft .NET, JavaScript e Java. Consulte [Microsoft Foundry SDK client libraries](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em preview ou em active development. Você pode encontrar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com sucesso com o Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [Microsoft Foundry portal](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar para a página inicial.

1. Se ainda não estiver habilitada, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para o seu projeto:
    - **Foundry resource**: *Use o nome padrão do seu recurso (geralmente {project_name}-resource)*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um resource group*
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade de modelos pode variar por região.

1. Selecione **Create**. Aguarde a criação do seu projeto.
1. Na página inicial do seu projeto, observe que a API key, o project endpoint e o OpenAI endpoint são exibidos aqui.

    > **Dica**: Você vai precisar do project endpoint mais tarde!

## Obter os arquivos do aplicativo no GitHub

Os arquivos iniciais do aplicativo de que você precisa para desenvolver o aplicativo de análise de avaliações são fornecidos em um repositório do GitHub.

1. Abra o Visual Studio Code.
1. Abra a Command Palette (*Ctrl+Shift+P*) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Talvez seja solicitado que você confirme que confia nos autores.

1. Depois que o repositório for clonado, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/01-analyze-text/Python/text-analysis**. Os arquivos do aplicativo incluem:
    - **reviews** (uma subpasta contendo os documentos de avaliação)
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **text-analysis.py** (o arquivo de código do aplicativo)

## Configurar seu aplicativo

1. No Visual Studio Code, veja o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente, se tiver um, ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.13.x.

    > **Dica**: Se for solicitado a instalar dependências, você pode instalar as do arquivo *requirements.txt* na pasta */Labfiles/01-analyze-text/Python/text-analysis*; mas tudo bem se não instalar agora — nós as instalaremos depois!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e, em seguida, ativá-lo com `\labenv\Scripts\activate`.

1. No painel **Explorer**, clique com o botão direito na pasta **text-analysis** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/01-analyze-text/Python/text-analysis*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **text-analysis** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o Azure Language Text Analytics SDK e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

1. No painel **Explorer**, na pasta **text-analysis**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o endpoint (até o domínio *.com*) do seu projeto Foundry (copie-os do portal do Foundry).

    > **Importante**: Modifique o endpoint colado para remover o sufixo "/api/projects/{project_name}" — o endpoint deve ser *https://{your-foundry-resource-name}.services.ai.azure.com*.

    Salve o arquivo de configuração modificado.

## Adicionar código para se conectar ao seu recurso Azure AI Language

1. No painel **Explorer**, na pasta **text-analysis**, abra o arquivo **text-analysis.py**.
1. Revise o código existente. Você adicionará código para trabalhar com o Azure Language Text Analytics SDK.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces que você precisará para usar o Text Analytics SDK:

    ```python
   # import namespaces
   from azure.identity import DefaultAzureCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Na função **main**, observe que já foi fornecido o código para carregar o endpoint do arquivo de configuração. Em seguida, localize o comentário **Create client using endpoint** e adicione o seguinte código para criar um client para a Text Analysis API:

    ```Python
   # Create client using endpoint
   credential = DefaultAzureCredential()
   ai_client = TextAnalyticsClient(endpoint=foundry_endpoint, credential=credential)
    ```

1. Salve as alterações no arquivo de código. Em seguida, no painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, usar apenas *az login* será suficiente. No entanto, se você tiver assinaturas em vários tenants, talvez seja necessário especificar o tenant usando o parâmetro *--tenant*. Consulte [Entrar no Azure de forma interativa usando o Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu recurso Foundry.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python text-analysis.py
    ```

1. Observe a saída, pois o código deve ser executado sem erro, exibindo o conteúdo de cada arquivo de texto de avaliação na pasta **reviews**. O aplicativo cria com sucesso um client para a Text Analytics API, mas ainda não o utiliza. Vamos corrigir isso na próxima seção.

## Adicionar código para detectar idioma

Agora que você criou um client para a API, vamos usá-lo para detectar o idioma em que cada avaliação foi escrita.

1. No editor de código, localize o comentário **Get language**. Em seguida, adicione o código necessário para detectar o idioma em cada documento de avaliação:

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Observação**: *Neste exemplo, cada avaliação é analisada individualmente, resultando em uma chamada separada ao serviço para cada arquivo. Uma abordagem alternativa é criar uma coleção de documentos e passá-los ao serviço em uma única chamada. Em ambas as abordagens, a response do serviço consiste em uma coleção de documentos; por isso, no código Python acima, o índice do primeiro (e único) documento na response ([0]) é especificado.*

1. Salve suas alterações. Em seguida, execute o programa novamente.
1. Observe a saída, observando que desta vez o idioma de cada avaliação é identificado.

## Adicionar código para extrair entities

Com frequência, documentos ou outros corpos de texto mencionam pessoas, lugares, períodos de tempo ou outras entities. A Text Analytics API pode detectar várias categorias (e subcategorias) de entity no seu texto.

1. No editor de código, localize o comentário **Get entities**. Em seguida, adicione o código necessário para identificar as entities mencionadas em cada avaliação:

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Salve suas alterações e execute o programa novamente.
1. Observe a saída, observando as entities que foram detectadas no texto.

## Adicionar código para PII redaction

Com frequência, políticas de privacidade e legislações podem exigir que personally identifiable information (PII), como nomes, endereços, números de telefone e outros detalhes privados, seja redigida dos documentos.

1. No editor de código, localize o comentário **Get PII**. Em seguida, adicione o código necessário para identificar entities de PII mencionadas em cada avaliação:

    ```python
   # Get PII
   pii_result = ai_client.recognize_pii_entities(documents=[text])[0]
   pii_entities = pii_result.entities
   if len(pii_entities) > 0:
        print("\nPII Entities")
        for pii_entity in pii_entities:
            print('\t{} ({})'.format(pii_entity.text, pii_entity.category)) 
        print("Redacted Text:\n {}".format(pii_result.redacted_text))
    ```

1. Salve suas alterações e execute o programa novamente.
1. Observe a saída, observando as PII entities que são identificadas e revisando a versão redigida de cada documento que é produzida.

## Limpar

Se você terminou de explorar o Azure Language no Foundry Tools, deverá excluir os recursos que criou neste exercício para evitar incorrer em custos desnecessários do Azure.

1. Abra o [Azure portal](https://portal.azure.com) e visualize o conteúdo do resource group onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do resource group e confirme que deseja excluí-lo.
