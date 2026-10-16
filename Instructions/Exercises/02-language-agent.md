---
lab:
    title: 'Desenvolver um agente de análise de texto'
    description: 'Use Azure Language in Foundry Tools para adicionar recursos de análise de texto a um agente de IA.'
    duration: 30
    level: 300
    islab: true
    status: 'released'
layout: default
---

# Desenvolver um agente de análise de texto

O **Azure Language in Foundry Tools** dá suporte à análise de texto, incluindo detecção de idioma, reconhecimento de entidades e redação de PII.

Você pode usar o serviço diretamente em um aplicativo por meio de sua REST API e de vários SDKs específicos de linguagem. Você também pode usar o **Azure Language in Foundry Tools MCP server** para integrar seus recursos a um agente de IA, que é o que você fará neste exercício.

O código usado neste exercício é baseado no Microsoft Foundry SDK for Python. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte [Microsoft Foundry SDK client libraries](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode experimentar algum comportamento inesperado, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalada

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com êxito com Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [Microsoft Foundry portal](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez em que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar até a home page.

1. Se ainda não estiver habilitada, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se for solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para o seu projeto:
    - **Foundry resource**: Use o nome padrão para o seu recurso (geralmente {project_name}-resource)
    - **Subscription**: Sua assinatura do Azure
    - **Resource group**: Crie ou selecione um grupo de recursos
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade de modelos pode variar por região.

    > **Dica**: Lembre-se (ou anote) o nome do Foundry resource — você vai precisar dele mais tarde!

1. Selecione **Create**. Aguarde a criação do seu projeto.
1. Na home page do seu projeto, observe que a API key, o project endpoint e o OpenAI endpoint são exibidos aqui.

    > **Dica**: Copie a project key para a área de transferência — você vai precisar dela mais tarde!

## Criar um agente

Agora que você tem um projeto do Foundry, pode criar um agente.

1. Agora você está pronto para **Start building**. Selecione **Create agents** (ou na página **Build**, selecione a guia **Agents**); e crie um novo agente chamado `Text-Analysis-Agent`.

    Quando estiver pronto, seu agente será aberto no playground do agente.

1. Na lista suspensa de modelos, verifique se um modelo **gpt-5** foi implantado e selecionado para o seu agente.
1. Atribua as seguintes **Instructions** ao seu agente:

    ```
   You are an AI agent that assists users by helping them analyze text.
    ```

1. Use o botão **Save** para salvar as alterações.
1. Teste o agente inserindo o seguinte prompt no painel **Chat**:

    ```
   What can you help me with?
    ```

    O agente deve responder com uma resposta apropriada com base em suas instruções.

## Criar uma conexão do Azure Language in Foundry Tools

O Foundry inclui um MCP server para o Azure Language in Foundry Tools, que você pode conectar ao seu projeto e usar no seu agente.

1. No painel de navegação à esquerda, selecione a página **Tools**.
1. Dentro da página Tools, selecione a guia **Tools**.
1. Conecte uma ferramenta; selecionando **Azure Language in Foundry Tools** no **Catalog** e conectando-a a um endpoint, especificando a seguinte configuração
    - **Name**: Um nome exclusivo para sua ferramenta
    - **Remote MCP Server endpoint**: `https://{foundry-resource-name}.cognitiveservices.azure.com/language/mcp?api-version=2025-11-15-preview`
    - **Parameters**: foundry-resource-name: Seu foundry resource name
    - **Authentication**: Key-based:
    - **Ocp-Apim-Subscription-Key**: API Key para o seu projeto do Foundry

    > **Observação**: Se a autenticação baseada em chave estiver desabilitada por uma política na sua assinatura do Azure, você pode usar a autenticação do Entra ID para conectar o agente ao serviço Azure Language.

1. Aguarde a criação da conexão da ferramenta MCP e, em seguida, veja a página de detalhes dela.
1. Na página de detalhes da conexão do Azure Language in Foundry Tools, selecione **Use in an agent** e, em seguida, selecione o agente **Text-Analysis-Agent** que você criou anteriormente.

    O agente deve ser aberto no playground, com a ferramenta Azure Language in Foundry Tools conectada.

    > **Observação**: Em alguns casos, a ferramenta MCP não é adicionada ao agente automaticamente. Verifique se **Azure Language in Foundry Tools** aparece na lista de ferramentas do agente. Se não aparecer, adicione-a manualmente a partir das ferramentas disponíveis para o agente.

## Testar a ferramenta Azure Language no playground

Agora vamos testar a capacidade do agente de usar a ferramenta que você conectou.

1. No playground do agente para o agente **Text-Analysis-Agent**, modifique as instruções da seguinte forma:

    ```
   You are an AI agent that assists users by helping them analyze text. Use the Azure Language tool to perform text analysis tasks.
    ```

1. Use o botão **Save** para salvar as alterações.
1. Teste o agente inserindo o seguinte prompt no painel **Chat**:

    ```
    Identify the PII entities in this article, and generate a redacted version:

    Microsoft was founded on April 4, 1975, by childhood friends Bill Gates (then 19) and Paul Allen (22) after they were inspired by the Altair 8800, one of the first personal computers, featured on the cover of Popular Electronics. They contacted the Altair’s maker, MITS, and successfully developed a version of the BASIC programming language, despite initially not owning the machine themselves. The pair formed a partnership called “Micro‑Soft” in Albuquerque, New Mexico, close to MITS’s headquarters, with the goal of writing software for emerging microcomputers.
    
    In the late 1970s, Microsoft grew by supplying programming languages to multiple hardware vendors, then relocated to the Seattle area in 1979. A pivotal moment came in 1980 when Microsoft partnered with IBM to provide an operating system for the IBM PC, leading to MS‑DOS and establishing the company’s dominance in personal computing. Gates guided the company’s long-term strategy as CEO, while Allen contributed key technical vision in its early years, setting Microsoft on a path that would reshape the software industry.
    ```

1. Quando solicitado, aprove o uso da ferramenta Azure Language selecionando **Always approve all Azure Language in Foundry Tools tools** (talvez você precise fazer isso duas vezes porque o prompt solicitou duas tarefas de análise de texto distintas).

    > **Observação**: Dependendo do seu ambiente, ocasionalmente você pode receber um erro transitório 403, 502 ou outro após aprovar a ferramenta. Esses erros geralmente não são bloqueadores. Atualize a página e tente o mesmo prompt novamente.

1. Revise a resposta, que deve identificar quaisquer informações de identificação pessoal no artigo sobre a fundação da Microsoft e criar uma versão do artigo com essas informações redigidas.
1. Revise os **Logs** do chat e verifique se a ferramenta Azure Language foi usada pelo agente para processar o prompt.

## Configurar a aprovação da ferramenta

Como você viu no playground, para usar a ferramenta, o agente precisa de aprovação.

1. No playground, na lista de **Tools** sob **Instructions**, no menu da ferramenta Azure Language que você adicionou, selecione **Configure**.
1. Verifique se a configuração **Approval setting for tools in this MCP server for this agent** está como **Always auto-approve all tools** (se não estiver, altere e adicione).
1. Salve quaisquer alterações no agente.

## Criar um aplicativo cliente

Agora que você tem um agente funcional, pode criar um aplicativo cliente que o use.

### Obter os arquivos do aplicativo no GitHub

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (Ctrl+Shift+P) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Talvez seja solicitado que você confirme que confia nos autores.

1. Depois que o repositório for clonado, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/02-language-agent/Python/text-agent**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **text-agent.py** (o arquivo de código do aplicativo)

### Configurar o aplicativo

1. No Visual Studio Code, veja o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente se você tiver um ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.1x.

    > **Dica**: Se for solicitado que você instale dependências, você pode instalar as que estão no arquivo *requirements.txt* na pasta */Labfiles/02-language-agent/Python/text-agent*; mas não tem problema se você não fizer isso agora, nós as instalaremos depois.

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e depois ativá-lo com `\labenv\Scripts\activate`.

1. No painel **Explorer**, clique com o botão direito na pasta **text-agent** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/02-language-agent/Python/text-agent*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez você precise habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **text-agent** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o pacote do Foundry SDK, o pacote Azure Identity e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

1. No painel **Explorer**, na pasta **text-agent**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o seu **endpoint** do projeto (na home page do projeto no Foundry Portal) e o nome do seu agente (que deve ser **Text-Analysis-Agent** — observe que esse nome diferencia maiúsculas de minúsculas).
1. Salve o arquivo de configuração modificado.

### Implementar o código do aplicativo

1. No painel **Explorer**, na pasta **text-agent**, abra o arquivo **text-agent.py**.
1. Veja o código existente. Você adicionará código para enviar prompts ao seu agente.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. No topo do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces de que você precisará:

    ```python
   # import namespaces
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. Na função **main**, observe que já foi fornecido código para carregar o endpoint do arquivo de configuração. Em seguida, localize o comentário **Get project client** e adicione o seguinte código para criar um cliente para o seu projeto do Foundry:

    ```python
   # Get project client
   project_client = AIProjectClient(
        endpoint=foundry_endpoint,
        credential=DefaultAzureCredential(),
   )
    ```

1. Localize o comentário **Get an OpenAI client** e adicione o seguinte código para obter um cliente do OpenAI com o qual chamar seu agente.

    ```python
   # Get an OpenAI client
   openai_client = project_client.get_openai_client()
    ```

1. Localize o comentário **Use the agent to get a response** e adicione o seguinte código para enviar um prompt do usuário ao seu agente e exibir a resposta.

    ```python
   # Use the agent to get a response
   prompt = input("User prompt: ")
   response = openai_client.responses.create(
        input=[{"role": "user", "content": prompt}],
        extra_body={"agent_reference": {"name": agent_name, "type": "agent_reference"}},
   )

   print(f"{agent_name}: {response.output_text}")
    ```

1. Salve as alterações feitas no arquivo de código.

## Testar o aplicativo cliente

Agora vamos testar o aplicativo executando-o em um ambiente Python e autenticando a conexão com o seu projeto.

1. No terminal do Visual Studio Code, insira o seguinte comando para entrar no Azure

   ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu recurso do Foundry.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```powershell
    python text-agent.py
    ```

1. Quando solicitado, insira o seguinte prompt:

    ```
    Extract named entities from the following text: "Pierre and I went to Paris on July 14th."
    ```

1. Revise a resposta, que deve identificar pessoas, locais e datas nomeadas.

## Ver detalhes da ferramenta

A ferramenta Azure Language in Foundry Tools oferece uma ampla variedade de funcionalidades, e o agente deve selecionar a função apropriada a ser chamada. Podemos ver as opções disponíveis na resposta do agente.

1. No arquivo de código **text-agent.py**, adicione a seguinte linha imediatamente após a linha *print(f"{agent_name}: {response.output_text}")* que você adicionou anteriormente (antes da linha *except Exception as ex:*):

    ```python
    print(f"\nResponse Details: {response.model_dump_json(indent=2)}")
    ```

1. Salve as alterações no arquivo de código.
1. No terminal, insira novamente o comando para executar o aplicativo (`python text-agent.py`).
1. Quando solicitado, insira o seguinte comando:

    ```
    Tell me what entities and dates are mentioned in this review, and whether it is positive or negative: "I booked my flight to Paris in July with Margie's Travel, and it was fantastic!"
    ```

1. Revise a resposta (talvez seja necessário rolar bastante para cima para vê-la), que deve identificar entidades e datas e determinar o sentimento do texto.
1. Revise os detalhes da resposta em JSON, que indicam cada uma das ferramentas disponíveis para o agente. Nesse caso, ele deve ter usado as ferramentas **extract_named_entities_from_text** e **detect_sentiment_from_text** dentro do Azure Language in Foundry Tools.

## Limpar recursos

Se você terminou de explorar o serviço Azure Language, poderá excluir os recursos criados neste exercício. Veja como:

1. No portal do Azure, navegue até o Foundry resource que você criou neste laboratório.
1. Na página do recurso, selecione **Delete** e siga as instruções para excluir o recurso.
