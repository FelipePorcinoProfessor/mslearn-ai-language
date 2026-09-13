---
lab:
    title: 'Usar Azure Speech em um agente'
    description: Use o servidor MCP do Azure Speech no Foundry Tools para adicionar recursos de fala a um agente.
    duration: 30
    level: 300
    islab: true
    status: 'released'
---

# Usar Azure Speech em um agente

O **Azure Speech no Foundry Tools** fornece um servidor MCP que você pode usar para permitir que um agente chame seus recursos de reconhecimento e síntese de fala.

Neste exercício, você vai configurar o servidor MCP do Azure Speech no Foundry Tools e conectá-lo a um agente.

O código usado neste exercício é baseado no Microsoft Foundry SDK for Python. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte [Microsoft Foundry SDK client libraries](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode enfrentar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com êxito com o Python 3.13.12.

## Criar uma conta de armazenamento do Azure

O servidor MCP do Azure Speech usa uma conta de armazenamento do Azure para salvar arquivos de áudio gerados.

1. Abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e entre com suas credenciais do Azure.
1. Crie um novo recurso **Azure storage account** com as seguintes configurações:
    - **Subscription**: *Sua assinatura*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Storage account name**: *Um nome exclusivo para sua conta de armazenamento*
    - **Region**: *Qualquer região disponível*
    - **Preferred storage type**: Azure blob storage ou Azure Data Lake Storage Gen2
    - **Primary workload**: Cloud native
    - **Performance**: Standard
    - **Redundancy**: Locally-redundant storage (LRS)
1. Quando o recurso da conta de armazenamento do Azure for criado, acesse-o no portal.
1. No painel de navegação esquerdo da conta de armazenamento, expanda **Data storage** e selecione **Containers**.
1. Adicione um novo contêiner chamado **files**. É aqui que seu agente salvará os arquivos de áudio que gerar.
1. No menu de contexto (**...**) do contêiner **files**, selecione **Generate SAS** e crie um token SAS com os seguintes detalhes:
    - **Signing method** : Account key
    - **Signing key**: Key1
    - **Stored access policy**: None
    - **Permissions**:
        - Read
        - Add
        - Create
        - Write
        - List
    - **Start and expiry date/time**:
        - **Start**: A data e hora atuais
        - **Expiry**: 23:59 de amanhã
    - **Allowed IP addresses**: *Deixe em branco*
    - **Allowed protocols**: HTTPS only

    > **IMPORTANTE**: Copie o token e a URL SAS gerados e armazene-os em um arquivo de texto por enquanto — você vai precisar deles mais tarde!

## Criar um projeto Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da web, abra o [portal Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez em que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar até a página inicial.

1. Se ainda não estiver habilitada, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para seu projeto:
    - **Foundry resource**: *Use o nome padrão do seu recurso (geralmente {project_name}-resource)*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade do modelo pode variar por região.

    > **DICA**: Lembre-se (ou anote) o nome do recurso Foundry — você vai precisar dele mais tarde!

1. Aguarde a criação do seu projeto.
1. Na página inicial do seu projeto, observe que a chave de API, o endpoint do projeto e o OpenAI endpoint são exibidos aqui.

    > **DICA**: Copie a chave do projeto para a área de transferência — você vai precisar dela mais tarde!

## Criar um agente

Agora que você tem um projeto Foundry, pode criar um agente.

1. Agora você está pronto para **Start building**. Selecione **Create agents** (ou na página **Build**, selecione a guia **Agents**); e crie um novo agente chamado `speech-agent`.

     Quando estiver pronto, seu agente será aberto no playground do agente.

1. Na lista suspensa de modelos, verifique se um modelo **gpt-5** foi implantado e selecionado para o seu agente.
1. Atribua ao seu agente as seguintes **Instructions**:

    ```
    You are an AI agent that uses the Azure AI Speech tool to transcribe and generate speech.
    ```

1. Use o botão **Save** para salvar as alterações.
1. Teste o agente inserindo o seguinte prompt no painel **Chat**:

    ```
   What can you help me with?
    ```

    O agente deve responder com uma resposta apropriada com base em suas instruções.

## Criar uma conexão do Azure Speech no Foundry Tools

O Foundry inclui um servidor MCP para Azure Speech no Foundry Tools, que você pode conectar ao seu projeto e usar no seu agente.

1. No painel de navegação à esquerda, selecione a página **Tools**.
1. Na guia **Tools**, conecte uma ferramenta; selecionando **Azure Speech MCP Server** no **Catalog** e conectando-a a um endpoint, especificando a seguinte configuração
    - **Name**: *Um nome exclusivo para sua ferramenta.*
    - **Remote MCP Server endpoint**: `https://{foundry-resource-name}.cognitiveservices.azure.com/speech/mcp?api-version=2025-11-15-preview`
    - **Parameters**: foundry-resource-name: *O nome do seu recurso Foundry*
    - **Authentication**: Key-based:
    - **Ocp-Apim-Subscription-Key**: *API Key do seu projeto Foundry*

    - **X-Blob-Container-Url**: *A URL SAS do seu contêiner de armazenamento*

    > **Observação**: Se a autenticação baseada em chave estiver desabilitada por uma política na sua assinatura do Azure, você pode usar a autenticação do Entra ID para conectar o agente ao serviço Azure Speech.

1. Aguarde a criação da conexão da ferramenta MCP e, em seguida, visualize sua página de detalhes.
1. Na página de detalhes da conexão do Azure Speech no Foundry Tools, selecione **Use in an agent** e, em seguida, selecione o agente **Speech-Agent** que você criou anteriormente.

    O agente deve ser aberto no playground, com a ferramenta Azure Speech no Foundry Tools conectada.

## Testar a ferramenta Azure Speech no playground

Agora vamos testar a capacidade do agente de usar a ferramenta que você conectou.

1. No playground do agente para o agente **speech-agent**, insira o seguinte prompt:

    ```
    Generate "To be or not to be, that is the question." as speech
    ```

1. Quando for solicitado, aprove o uso da ferramenta Azure Speech selecionando **Always approve all Azure Speech MCP Server tools**.

1. Revise a resposta, que deve incluir um link para o arquivo de áudio gerado. Em seguida, clique no link para ouvir a fala sintetizada.
1. Insira o seguinte prompt:

    ```
    Transcribe the file at https://microsoftlearning.github.io/mslearn-ai-language/Labfiles/05-speech-tool/speech_1.wav
    ```

1. Se solicitado, aprove o uso da ferramenta Azure Speech selecionando **Always approve all Azure Speech MCP Server tools**.

    > **Observação**: Se ocorrer um erro transitório ao chamar a ferramenta, atualize a página e tente o mesmo prompt novamente.

    > **Observação**: Você pode ver uma mensagem informando que a interação foi bloqueada por um Foundry guardrail. Isso acontece porque as verificações internas de segurança de conteúdo do Foundry sinalizam o conteúdo transcrito de `speech_1.wav`. Se isso ocorrer, tente novamente o prompt usando `speech_2.wav`, que não é afetado.

1. Revise a saída, que deve ser uma transcrição do arquivo de áudio.

## Criar um aplicativo cliente

Agora que você tem um agente funcional, pode criar um aplicativo cliente que o utiliza.

### Obter os arquivos do aplicativo no GitHub

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (*Ctrl+Shift+P*) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Pode ser solicitado que você confirme se confia nos autores.

1. Depois que o repositório for clonado, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/05-speech-tool/Python/speech-client**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **speech-client.py** (o arquivo de código do aplicativo)

### Configurar o aplicativo

1. No Visual Studio Code, exiba o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente se você tiver um ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.1x.

    > **Dica**: Se você for solicitado a instalar dependências, poderá instalar as do arquivo *requirements.txt* na pasta */Labfiles/05-speech-tool/Python/speech-client*; mas tudo bem se você não o fizer — vamos instalá-las mais tarde!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e depois ativá-lo com `\labenv\Scripts\activate`.

1. No painel **Explorer**, clique com o botão direito do mouse na pasta **speech-client** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/05-speech-tool/Python/speech-client*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Pode ser necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **speech-client** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o pacote Foundry SDK, o pacote Azure Identity e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

1. No painel **Explorer**, na pasta **speech-client**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o **endpoint** do seu projeto (na página inicial do projeto no Foundry Portal) e o nome do seu agente (que deve ser **speech-agent** — observe que esse nome diferencia maiúsculas de minúsculas).
1. Salve o arquivo de configuração modificado.

### Implementar o código do aplicativo

1. No painel **Explorer**, na pasta **speech-client**, abra o arquivo **speech-client.py**.
1. Revise o código existente. Você adicionará código para enviar prompts ao seu agente.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. No início do arquivo de código, abaixo das referências de namespace existentes, encontre o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces de que você precisa:

    ```python
   # import namespaces
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. Na função **main**, observe que o código para carregar o endpoint do arquivo de configuração já foi fornecido. Em seguida, encontre o comentário **Get project client** e adicione o seguinte código para criar um cliente para o seu projeto Foundry:

    ```python
   # Get project client
   project_client = AIProjectClient(
        endpoint=foundry_endpoint,
        credential=DefaultAzureCredential(),
   )
    ```

1. Encontre o comentário **Get an OpenAI client** e adicione o seguinte código para obter um cliente OpenAI com o qual chamar seu agente.

    ```python
   # Get an OpenAI client
   openai_client = project_client.get_openai_client()
    ```

1. Encontre o comentário **Use the agent to get a response** e adicione o seguinte código para enviar um prompt do usuário ao seu agente e exibir a resposta.

    ```python
   # Use the agent to get a response
   response = openai_client.responses.create(
        input=[{"role": "user", "content": prompt}],
        extra_body={"agent_reference": {"name": agent_name, "type": "agent_reference"}},
   )

   print(f"{agent_name}: {response.output_text}")
    ```

1. Salve as alterações feitas no arquivo de código.

## Testar o aplicativo cliente

Agora vamos testar o aplicativo executando-o em um ambiente Python e autenticando a conexão com o seu projeto.

1. No painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, usar apenas *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu recurso Foundry.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```powershell
    python speech-client.py
    ```

1. Quando solicitado, insira o seguinte prompt:

    ```
    Synthesize "Better a witty fool, than a foolish wit!" as speech using the voice "en-GB-SoniaNeural".
    ```

1. Revise a resposta, que deve incluir um link clicável para um arquivo de áudio gerado.
1. Depois de verificar o arquivo de áudio gerado, insira o seguinte prompt:

     ```
     Transcribe https://microsoftlearning.github.io/mslearn-ai-language/Labfiles/05-speech-tool/speech_2.wav
     ```

1. Revise a resposta.
1. Para sair do programa, digite "quit" (ou apenas pressione Enter)

## Limpar recursos

Se você terminou de explorar o serviço Azure AI Language, pode excluir os recursos que criou neste exercício. Veja como:

1. No portal do Azure, navegue até o recurso Foundry que você criou neste laboratório.
1. Na página do recurso, selecione **Delete** e siga as instruções para excluir o recurso.
