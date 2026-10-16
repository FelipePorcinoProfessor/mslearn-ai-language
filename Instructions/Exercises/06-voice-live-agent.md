---
lab:
    title: 'Desenvolver um agente Voice Live'
    description: 'Use Azure Speech Voice Live no Microsoft Foundry Tools para criar um agente conversacional.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
layout: default
---

# Desenvolver um agente Voice Live

Agentes de IA com recursos de fala permitem que os usuários interajam de forma conversacional — usando comandos e perguntas faladas que geram respostas por voz.

Neste exercício, você usará o recurso Voice Live do Azure Speech no Microsoft Foundry Tools para criar um agente de voz em tempo real.

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode experimentar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de começar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalada

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com êxito com o Python 3.13.12.

## Criar um projeto do Microsoft Foundry

Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar até a página inicial.

1. Se ainda não estiver habilitado, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se for solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para o seu projeto:
    - **Foundry resource**: *Insira um nome válido para o seu recurso AI Foundry.*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade de modelos e recursos pode variar por região.

1. Selecione **Create**. Aguarde a criação do seu projeto. Em seguida, exiba a página inicial dele.

## Criar um agente

Agora vamos criar um agente.

1. Agora você está pronto para **Start building**. Selecione **Create agents** (ou, na página **Build**, selecione a guia **Agents**) e crie um novo agente chamado `chat-agent`.

     Quando estiver pronto, seu agente será aberto no playground do agente.

1. Na lista suspensa de modelos, verifique se um modelo **gpt-5** foi implantado e selecionado para o seu agente.
1. Atribua ao seu agente as seguintes **Instructions**:

    ```
   You are an AI assistant that helps people find information about AI and related topics. You answer questions concisely and precisely.
    ```

1. Use o botão **Save** para salvar as alterações.
1. Teste o agente inserindo o seguinte prompt no painel **Chat**:

    ```
   What can you help me with?
    ```

    O agente deve responder com uma resposta apropriada com base nas instruções.

## Configurar Azure Speech Voice Live

Habilitar o modo de fala para um agente do Foundry integra Azure Speech Voice Live — adicionando recursos de fala ao agente.

1. No painel à esquerda, abaixo da lista de seleção do modelo, habilite **Voice mode**.

    Se o painel **Configuration** não abrir automaticamente, use o ícone de engrenagem acima da interface de chat para abri-lo.

1. No painel **Configuration**, em **Voice Live**, revise a configuração padrão de entrada e saída de fala. Você pode experimentar vozes diferentes, pré-visualizando-as até decidir qual usar.
1. Feche o painel **Configuration** e use o botão **Save** para salvar o agente.

## Usar fala para interagir com o agente

Agora você está pronto para conversar com o agente.

1. No painel Chat, use o botão **Start session** para iniciar uma conversa com o agente. Se solicitado, permita o acesso ao microfone do sistema.

    O agente iniciará uma sessão de fala e ouvirá o seu prompt.

1. Quando o status do aplicativo for **Listening…**, diga algo como "*Como funciona o reconhecimento de fala?*" e aguarde uma resposta.

1. Verifique se o status do aplicativo muda para **Processing…**. O aplicativo processará a entrada falada.

    >**Dica**: A velocidade de processamento pode ser tão rápida que você pode não ver o status antes de ele voltar a *Speaking*.

1. Quando o status mudar para **Speaking…**, o aplicativo usará conversão de texto em fala para vocalizar a resposta do modelo. Para ver o prompt original e a resposta como texto, selecione o botão **cc** na parte inferior da tela de chat.

    >**Dica**: O prompt de acompanhamento é enviado apenas falando. Você pode até interromper o agente para manter a interação focada no que você precisa fazer. Você também pode usar o botão **Stop generation** no painel de chat para interromper respostas de longa duração. O botão encerrará a conversa. Você precisará iniciar uma nova conversa para continuar usando o agente.

1. Para continuar a conversa, basta fazer outra pergunta, como "*Como funciona a síntese de fala?*", e revisar a resposta.
1. Quando terminar de conversar com o agente, use o ícone **X** para encerrar a sessão. Uma transcrição da conversa será exibida.

## Criar um aplicativo cliente

Para usar seu agente em um aplicativo personalizado, você precisa escrever código que use o Azure Speech Voice Live SDK para iniciar e gerenciar uma sessão de conversa.

### Obter os arquivos do aplicativo no GitHub

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (*Ctrl+Shift+P*) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Pode ser solicitado que você confirme se confia nos autores.

1. Depois que o repositório for clonado, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/06-voice-live/Python/chat-client**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **chat-client.py** (o arquivo de código do aplicativo)

### Configurar o aplicativo

1. No Visual Studio Code, veja o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente se você tiver um ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.13.xx.

    > **Dica**: Se for solicitado que você instale dependências, você pode instalar as que estão no arquivo *requirements.txt* na pasta */Labfiles/06-voice-live/Python/chat-client*; mas tudo bem se você não fizer isso — vamos instalá-las depois!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e, depois, ativá-lo com `\labenv\Scripts\activate`.

1. No painel **Explorer**, clique com o botão direito na pasta **chat-client** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/06-voice-live/Python/chat-client*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **chat-client** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o pacote do Foundry SDK, o pacote Azure Identity e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt azure-identity azure-ai-voicelive==1.2.0b4 --pre azure-ai-projects==2.0.0b4
    ```

1. No painel **Explorer**, na pasta **chat-client**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para incluir o **endpoint** do seu recurso Foundry (obtenha o endpoint do projeto na página inicial do projeto no Foundry Portal, mas use apenas a URL base até o domínio *.com*), o nome do seu projeto e o nome do seu agente (que deve ser **Chat-Agent** — observe que esse nome diferencia maiúsculas de minúsculas).

> **Importante**: Modifique o endpoint colado para remover o sufixo "/api/projects/{project_name}" — o endpoint deve ser *https://{your-foundry-resource-name}.services.ai.azure.com*.

1. Salve o arquivo de configuração modificado.

### Implementar o código do aplicativo

1. No painel **Explorer**, na pasta **chat-client**, abra o arquivo **chat-client.py**.
1. Revise o código existente. A maior parte da estrutura do aplicativo foi fornecida — você deve implementar as etapas principais necessárias para usar o Voice Live SDK a fim de gerenciar uma conversa com seu agente.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. Na parte superior do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces de que você precisará:

    ```python
   # import namespaces
   from azure.identity.aio import AzureCliCredential
   from azure.ai.voicelive.aio import connect
   from azure.ai.voicelive.models import (
        InputAudioFormat,
        Modality,
        OutputAudioFormat,
        RequestSession,
        ServerEventType,
        AudioNoiseReduction,
        AudioEchoCancellation,
        AzureSemanticVadMultilingual,
        AgentConfig
   ) 
    ```

1. Na função **main**, observe que já foi fornecido o código para carregar o endpoint do arquivo de configuração, bem como o código para obter uma credencial de autenticação e para criar e executar um objeto **VoiceAssistant**.

    A classe **VoiceAssistant** encapsula a lógica para gerenciar a conversa do Voice Live.

1. Abaixo da função **main**, localize a definição da classe **VoiceAssistant**.

    A função ****init**** para inicializar um objeto baseado na classe já foi implementada.

    Você deve implementar a função **start**, que é a função principal para estabelecer a sessão de conversa.

1. Localize o comentário **STEP 1: Connect Azure VoiceLive to the agent** e adicione o seguinte código (tendo cuidado para indentá-lo um nível dentro da instrução **try:**):

    ```python
   # STEP 1: Connect Azure VoiceLive to the agent
   async with connect(
        endpoint=self.endpoint,
        credential=self.credential,
        api_version="2026-01-01-preview",
        agent_config=self.agent_config
   ) as connection:
        self.connection = connection
    ```

    Esta etapa cria uma conexão com o seu agente para que o Voice Live SDK possa estabelecer uma conversa com ele.

1. Localize o comentário **STEP 2: Initialize audio processor** e adicione o seguinte código (tendo cuidado para indentá-lo *mais um nível* dentro do código da etapa 1 que você acabou de adicionar):

    ```python
   # STEP 2: Initialize audio processor
   self.audio_processor = AudioProcessor(connection)
    ```

    Esse código anexa um objeto AudioProcessor com base na definição de classe mais abaixo no arquivo de código. AudioProcessor é uma classe utilitária para gerenciar E/S de hardware de áudio.

1. Localize o comentário **STEP 3: Configure the session** e adicione o seguinte código (tendo cuidado para manter a mesma indentação do código da etapa 2 acima):

    ```python
   # STEP 3: Configure the session
   await self.setup_session()
    ```

    Esse código configura a sessão com os formatos de áudio apropriados, semântica de detecção de alternância de fala e opções para lidar com ecos e ruído de fundo.

1. Localize o comentário **STEP 4: Start audio systems** e adicione o seguinte código (tendo cuidado para manter a mesma indentação do código da etapa 3 acima):

    ```python
   # STEP 4: Start audio systems
   self.audio_processor.start_playback()
            
   print("\n✅ Ready! Start speaking...")
   print("Press Ctrl+C to exit\n")
    ```

    Esse código inicia o processador de áudio para que ele monitore o microfone para entrada de áudio e reproduza a saída de áudio.

1. Localize o comentário **STEP 5: Process events** e adicione o seguinte código (tendo cuidado para manter a mesma indentação do código da etapa 4 acima):

    ```python
   # STEP 5: Process events
   await self.process_events()
    ```

    Esse código executa o loop principal para processar eventos como entrada de fala, saída de resposta e interrupções.

1. Salve as alterações no arquivo de código.

    A função concluída deve ficar assim:

    ```python
   async def start(self):
            """Start the voice assistant."""
            print("\n" + "=" *60)
            print(f"🎙️   {self.agent_config['agent_name']}")
            print("="* 60)
    
            # Add your code in this try block!
            try:
                # STEP 1: Connect Azure VoiceLive to the agent
                async with connect(
                    endpoint=self.endpoint,
                    credential=self.credential,
                    api_version="2026-01-01-preview",
                    agent_config=self.agent_config
                ) as connection:
                    self.connection = connection
                        
                    # STEP 2: Initialize audio processor
                    self.audio_processor = AudioProcessor(connection)
                                      
                    # STEP 3: Configure the session
                    await self.setup_session()
                    
                    # STEP 4: Start audio systems
                    self.audio_processor.start_playback()
            
                    print("\n✅ Ready! Start speaking...")
                    print("Press Ctrl+C to exit\n")
                    
                    # STEP 5: Process events
                    await self.process_events()
    
            finally:
                if hasattr(self, 'audio_processor'):
                    self.audio_processor.shutdown()
    ```

## Executar o aplicativo

Agora você está pronto para executar seu aplicativo e conversar com o seu agente.

> **DICA**: O aplicativo funciona melhor ao usar um headset. Ao usar alto-falantes, há o risco de o agente "ouvir" as próprias respostas e processá-las como nova entrada do usuário.

1. No terminal do Visual Studio Code, insira o seguinte comando para entrar no Azure

   ```powershell
    az login
    ```

    Quando solicitado, entre no Azure usando suas credenciais.

    > **Observação**: Na maioria dos cenários, usar apenas *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte *[Entrar no Azure interativamente usando o Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)* para obter detalhes.

1. No terminal do Visual Studio Code, confirme os detalhes da sua assinatura do Azure e, em seguida, insira o seguinte comando para executar o aplicativo cliente:

    ```powershell
    python chat-client.py
    ```

1. Quando solicitado, comece uma conversa com o agente fazendo uma pergunta como "*Como a fala computacional é usada em IA?*".
1. Ouça a resposta e então continue a conversa — observe que você pode interromper o agente para fazer novas perguntas.
1. Quando terminar, pressione **CTRL+C** para encerrar a conversa e parar o programa.

## Limpeza

Se você terminou de explorar o Microsoft Foundry, exclua quaisquer recursos de que não precise mais. Isso evita a cobrança de custos desnecessários.

1. Abra o **Azure portal** em [https://portal.azure.com](https://portal.azure.com) e selecione o grupo de recursos que contém os recursos que você criou.
1. Selecione **Delete resource group** e, em seguida, **enter the resource group name** para confirmar. O grupo de recursos será excluído.
