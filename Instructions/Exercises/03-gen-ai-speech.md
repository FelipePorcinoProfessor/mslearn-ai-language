---
lab:
    title: 'Use modelos de IA generativa com suporte a fala'
    description: Implemente funcionalidade de fala usando IA generativa.
    duration: 30
    level: 300
    islab: true
    status: 'released'
---

# Use modelos de IA generativa com suporte a fala

Cada vez mais, os recursos dos modelos de IA generativa estão evoluindo além do preenchimento de linguagem baseado em texto para oferecer suporte a conteúdo em outros formatos — incluindo fala audível.

Neste exercício, você usará modelos de IA generativa para dar suporte a dois cenários comuns:

- Síntese de fala (text-to-speech) — geração de saída de voz.
- Reconhecimento de fala (speech-to-text) — transcrição de entrada de voz.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de fala com IA generativa usando vários SDKs específicos de linguagem; incluindo:

- [OpenAI SDK for Python](https://pypi.org/project/openai/)
- [OpenAI SDK for .NET](<https://www.nuget.org/packages/OpenAI>)
- [OpenAI SDK for JavaScript](https://www.npmjs.com/package/openai)

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode encontrar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de começar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalada

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com sucesso no Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [Microsoft Foundry portal](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que forem abertos na primeira vez que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar até a página inicial.

1. Se ainda não estiver habilitado, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para o seu projeto:
    - **Foundry resource**: Use o nome padrão do recurso (geralmente {project_name}-resource)
    - **Subscription**: Sua assinatura do Azure
    - **Resource group**: Crie ou selecione um grupo de recursos
    - **Region**: Selecione East US 2 (Para este exercício, alguns modelos estão disponíveis somente nessa localização.)

1. Selecione **Create**. Aguarde a criação do seu projeto. Em seguida, exiba sua página inicial.

## Implantar modelos

Para desenvolver apps com suporte a fala, vamos precisar de modelos habilitados para fala. Especificamente, precisamos de um modelo que possa realizar geração de fala e um modelo que possa processar entrada de fala.

### Implantar um modelo de geração de fala

1. Agora você está pronto para **Start building**. Vamos **Explore models** (ou, na página **Discover**, selecione a guia **Models**) para ver o catálogo de modelos do Microsoft Foundry.
1. No catálogo de modelos, pesquise por `gpt-4o-mini-tts` ou outro modelo de text-to-speech disponível na sua região.
1. Revise o cartão do modelo e, em seguida, implante um modelo de text-to-speech disponível usando as configurações padrão.
1. Quando o modelo for implantado, exiba seus detalhes, observando que o **Target URI** e a **Key** necessários para usá-lo estão disponíveis aqui (você precisará do Target URI mais tarde).

    > **IMPORTANTE**: `gpt-4o-mini-tts` é um modelo recomendado para este exercício, não um requisito. Se ele não estiver disponível na sua região, implante um modelo de text-to-speech compatível.

### Implantar um modelo de reconhecimento de fala

1. Na barra de menus do Foundry portal, selecione **Build**; e depois exiba a página **Deployments**. Observe que o modelo de text-to-speech que você implantou está listado.
1. Selecione **Deploy a base model** e pesquise no catálogo por `gpt-4o-mini-transcribe` ou outro modelo de speech-to-text disponível na sua região.
1. Implemente um modelo de speech-to-text disponível usando as configurações padrão.
1. Volte à página **Deployments** e verifique se os dois modelos que você implantou estão listados.
1. Selecione qualquer um dos modelos para ver o Target URI que você precisa usar no seu código.

    > **IMPORTANTE**: `gpt-4o-mini-transcribe` é um modelo recomendado para este exercício, não um requisito. Se ele não estiver disponível na sua região, implante um modelo de speech-to-text compatível.

## Obter os arquivos do aplicativo no GitHub

Os arquivos iniciais do aplicativo de que você precisará para desenvolver aplicativos de fala são fornecidos em um repositório do GitHub.

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (Ctrl+Shift+P) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Depois, abra-o.

    Pode ser solicitado que você confirme que confia nos autores.

1. No Visual Studio Code, exiba o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente, se você tiver um, ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.1x.

    > **Dica**: Se você for solicitado a instalar dependências, você pode instalar as que estão no arquivo *requirements.txt* na pasta */Labfiles/03-gen-ai-speech/Python/generate-speech*; mas tudo bem se não instalar — nós as instalaremos mais tarde!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e ativá-lo com `\labenv\Scripts\activate`.

## Criar um aplicativo de geração de fala

1. Depois que o repositório for clonado, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/03-gen-ai-speech/Python/generate-speech**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **generate-speech.py** (o arquivo de código do aplicativo)

### Configurar seu aplicativo

1. No painel **Explorer**, clique com o botão direito do mouse na pasta **generate-speech** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/03-gen-ai-speech/Python/generate-speech*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **generate-speech** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o pacote OpenAI SDK e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

1. No painel **Explorer**, na pasta **generate-speech**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o **Target URI** (endpoint) do seu modelo **gpt-4o-mini-tts**.

    > **Dica**: Copie o Target URI da página de detalhes do modelo no Foundry portal.

    Salve o arquivo de configuração modificado.

### Escrever código para usar o modelo para geração de fala

1. No painel **Explorer**, na pasta **generate-speech**, selecione o arquivo **generate-speech.py** para abri-lo.
1. Revise o código existente. Você adicionará código para usar o OpenAI SDK para acessar seu modelo.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, encontre o comentário **Import namespaces** e adicione o seguinte código para importar o namespace necessário para usar o OpenAI SDK:

    ```python
   # import namespaces
   from openai import AzureOpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
    ```

1. Na função **main**, observe que o código para carregar o endpoint do arquivo de configuração já foi fornecido. Em seguida, encontre o comentário **Create the Azure OpenAI client** e adicione o seguinte código para criar um cliente para a API do OpenAI:

    ```Python
   # Create the Azure OpenAI client
   token_provider = get_bearer_token_provider(                    
        DefaultAzureCredential(), "https://ai.azure.com/.default"
    )

   client = AzureOpenAI(
        azure_endpoint=endpoint,
        azure_ad_token_provider = token_provider,
        api_version="2025-03-01-preview"
   )
    ```

1. Encontre o comentário **Generate speech and save to file** e adicione o seguinte código para enviar um prompt ao modelo de geração de fala e salvar a resposta como um arquivo.

    ```Python
   # Generate speech and save to file
   with client.audio.speech.with_streaming_response.create(
                model=model_deployment,
                voice="alloy",
                input="My voice is my passport!",
                instructions="Speak in a serious tone.",
            ) as response:
        response.stream_to_file(speech_file_path)
    ```

1. Salve as alterações no arquivo de código.

### Executar o aplicativo

1. No painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, apenas usar o comando az login será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro --tenant. Consulte [Entrar no Azure de forma interativa usando o Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, exibindo (e confirmando, se necessário) os detalhes da assinatura que contém seu recurso do Foundry.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python generate-speech.py
    ```

1. Observe a saída enquanto o código gera a fala solicitada e a salva em um arquivo. O código também deve reproduzir o arquivo de áudio gerado.

## Criar um aplicativo de transcrição de fala

1. No painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/03-gen-ai-speech/Python/transcribe-speech**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **transcribe-speech.py** (o arquivo de código do aplicativo)

### Configurar seu aplicativo

1. No painel **Explorer**, clique com o botão direito do mouse na pasta **transcribe-speech** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou, no terminal existente, navegue até a pasta */Labfiles/03-gen-ai-speech/Python/transcribe-speech*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **transcribe-speech** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou anteriormente está ativo.
1. Instale o pacote OpenAI SDK e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

    > **Observação**: Esta etapa na verdade não é necessária se você concluiu a parte anterior deste exercício, pois botg apps usam o mesmo ambiente e têm as mesmas dependências — mas não haverá problema em executá-la!

1. No painel **Explorer**, na pasta **transcribe-speech**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o **Target URI** (endpoint) do seu modelo **gpt-4o-mini-transcribe**.

    > **Dica**: Copie o Target URI da página de detalhes do modelo no Foundry portal.

    Salve o arquivo de configuração modificado.

### Escrever código para usar o modelo para transcrição de fala

1. No painel **Explorer**, na pasta **transcribe-speech**, selecione o arquivo **transcribe-speech.py** para abri-lo.
1. Revise o código existente. Você adicionará código para usar o OpenAI SDK para acessar seu modelo.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. Na parte superior do arquivo de código, sob as referências de namespace existentes, encontre o comentário **Import namespaces** e adicione o seguinte código para importar o namespace necessário para usar o OpenAI SDK:

    ```python
   # import namespaces
   from openai import AzureOpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
    ```

1. Na função **main**, observe que o código para carregar o endpoint do arquivo de configuração já foi fornecido. Em seguida, encontre o comentário **Create the Azure OpenAI client** e adicione o seguinte código para criar um cliente para a API do OpenAI:

    ```Python
   # Create the Azure OpenAI client
   token_provider = get_bearer_token_provider(                    
        DefaultAzureCredential(), "https://ai.azure.com/.default"
    )

   client = AzureOpenAI(
        azure_endpoint=endpoint,
        azure_ad_token_provider = token_provider,
        api_version="2025-03-01-preview"
   )
    ```

1. Encontre o comentário **Call model to transcribe audio file** e adicione o seguinte código para enviar um arquivo de áudio ao modelo de transcrição de fala e gerar uma transcrição.

    ```Python
   # Call model to transcribe audio file
   audio_file = open(file_path, "rb")
   transcription = client.audio.transcriptions.create(
        model=model_deployment,
        file=audio_file,
        response_format="text"
   )
        
   print(transcription)
        
    ```

1. Salve as alterações no arquivo de código.

### Executar o aplicativo

1. No painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, apenas usar o comando az login será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro --tenant. Consulte [Entrar no Azure de forma interativa usando o Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, exibindo (e confirmando, se necessário) os detalhes da assinatura que contém seu recurso do Foundry.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python transcribe-speech.py
    ```

1. Observe a saída enquanto o código envia o arquivo de áudio para o modelo para transcrição e exibe os resultados. O código também deve reproduzir o arquivo de áudio.

## Limpar recursos

Se você terminou de explorar modelos com suporte a fala no Foundry Tools, deverá excluir os recursos que criou neste exercício para evitar custos desnecessários no Azure.

1. Abra o [Azure portal](https://portal.azure.com) e exiba o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
