---
lab:
    title: 'Reconhecer e sintetizar fala'
    description: Implementar funcionalidade de fala usando Azure Speech in Foundry Tools.
    duration: 30
    level: 300
    islab: true
    status: 'released'
---

# Reconhecer e sintetizar fala

**Azure Speech in Foundry Tools** é um serviço que fornece funcionalidades relacionadas a fala, incluindo:

- Uma API de *speech-to-text* que permite implementar reconhecimento de fala (convertendo palavras faladas audíveis em texto).
- Uma API de *text-to-speech* que permite implementar síntese de fala (convertendo texto em fala audível).

Neste exercício, você usará ambas as APIs para implementar um assistente de mensagens de voz.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de fala usando vários SDKs específicos de linguagem, incluindo:

- [Azure Speech SDK for Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [Azure Speech SDK for .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [Azure Speech SDK for JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em prévia ou em desenvolvimento ativo. Você pode experimentar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de começar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com sucesso com Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [portal do Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre com suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar para a página inicial.

1. Se ainda não estiver habilitada, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se for solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para o seu projeto:
    - **Foundry resource**: *Use the default name for your resource (usually {project_name}-resource)*\*
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Create or select a resource group*
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade de modelos pode variar por região.

    > **Dica**: \* Lembre-se do nome do Foundry resource - você precisará dele mais tarde!

1. Aguarde a criação do seu projeto. Em seguida, visualize a página inicial do seu projeto.

## Obter os arquivos do aplicativo no GitHub

Os arquivos iniciais do aplicativo necessários para desenvolver o aplicativo de voz são fornecidos em um repositório do GitHub.

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (Ctrl+Shift+P) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Pode ser solicitado que você confirme que confia nos autores.

1. Após a clonagem do repositório, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/04-azure-speech/Python/voice-mail**. Os arquivos do aplicativo incluem:
    - **messages** (uma subpasta contendo gravações de áudio de mensagens)
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **voice-mail.py** (o arquivo de código do aplicativo)

## Configurar seu aplicativo

1. No Visual Studio Code, visualize o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente se você tiver um, ou crie um novo ambiente **Venv** baseado na sua instalação do Python 3.1x.

    > **Dica**: Se for solicitado que você instale dependências, você pode instalar as do arquivo *requirements.txt* na pasta */Labfiles/04-azure-speech/Python/voice-mail*; mas tudo bem se você não o fizer — vamos instalá-las mais tarde!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv`, depois ativá-lo com `\labenv\Scripts\activate`.

1. No painel **Explorer**, clique com o botão direito na pasta **voice-mail** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/04-azure-speech/Python/voice-mail*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **voice-mail** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o pacote Azure AI Speech SDK e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

1. No painel **Explorer**, na pasta **voice-mail**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para refletir o **endpoint** do Cognitive Services para o seu Foundry resource.

    > **Importante**: O endpoint deve ser *https://{YOUR_FOUNDRY_RESOURCE}.cognitiveservices.azure.com/*. O nome do Foundry Resource geralmente tem o formato *{project_name}-resource*.

    Salve o arquivo de configuração modificado.

## Adicionar código para sintetizar fala

1. No painel **Explorer**, na pasta **voice-mail**, abra o arquivo **voice-mail.py**.
1. Revise o código existente. Você adicionará código para trabalhar com o Azure Speech SDK.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. No topo do arquivo de código, sob as referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces que você precisará para usar o Speech SDK:

    ```python
   # import namespaces
   from azure.identity import DefaultAzureCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **main**, observe que o código para carregar o endpoint do arquivo de configuração já foi fornecido. Em seguida, localize o comentário **Create speech_config using Entra ID authentication** e adicione o seguinte código para criar um objeto Speech Configuration:

    ```Python
   # Create speech_config using Entra ID authentication
   credential = DefaultAzureCredential()
   speech_config = speech_sdk.SpeechConfig(    
        token_credential=credential,
        endpoint=foundry_endpoint)
    ```

1. Revise o restante da função **main** e observe que foi implementado um loop que permite ao usuário escolher uma de três opções:
    1. Gravar uma saudação de voz
    1. Transcrever mensagens
    1. Sair do aplicativo

1. Localize a função **record_greeting**, que você implementará para gravar uma saudação de voz como um arquivo de áudio.
1. Na função **record_greeting**, localize o comentário **Synthesize the greeting message to an audio file** e adicione o seguinte código para sintetizar fala a partir do texto inserido pelo usuário e salvá-la como um arquivo de áudio.

    ```python
   output_file = "greeting.wav"
   audio_config = speech_sdk.audio.AudioOutputConfig(filename=output_file)

   speech_config.speech_synthesis_voice_name = "en-US-Serena:DragonHDLatestNeural"

   speech_synthesizer = speech_sdk.SpeechSynthesizer(
        speech_config=speech_config,
        audio_config=audio_config
   )

   result = speech_synthesizer.speak_text_async(greeting_message).get()

   if result.reason == speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(f"Greeting recorded and saved to {output_file}")
        speech_synthesizer = None  # Release the synthesizer resources
   else:
        print("Error recording greeting: {}".format(result.reason))
    ```

1. Salve as alterações no arquivo de código. Em seguida, no painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, usar apenas o comando az login será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Entrar no Azure interativamente usando o Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu Foundry resource.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```powershell
   python voice-mail.py
    ```

1. Quando solicitado, insira **1** para gravar uma saudação.
1. Insira uma saudação, como `Hi. The person you called is not available right now. Leave a message.`
1. Aguarde enquanto a fala é sintetizada e salva como um arquivo de áudio.

    Você pode selecionar o arquivo *greeting.wav* que é gerado na pasta voice-mail para reproduzi-lo no Visual Studio Code.

## Adicionar código para reconhecer fala

1. No arquivo de código **voice-mail.py**, localize a função **transcribe_messages**, que você implementará para transcrever cada uma das mensagens de voz na subpasta **messages**.

    A função já contém código para percorrer os arquivos na pasta **messages**.

1. Na função **transcribe_messages**, localize o comentário **Transcribe the audio file** e adicione o seguinte código para transcrever o áudio.

    ```python
   # Transcribe the audio file
   audio_config = speech_sdk.audio.AudioConfig(filename=file_path)
   speech_recognizer = speech_sdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
   )
   result = speech_recognizer.recognize_once_async().get()
   if result.reason == speech_sdk.ResultReason.RecognizedSpeech:
        print(f"Transcription: {result.text}")
   else:
        print("Error transcribing message: {}".format(result.reason))
    ```

1. Salve as alterações no arquivo de código. Em seguida, no terminal, insira o seguinte comando para executar o aplicativo:

    ```powershell
   python voice-mail.py
    ```

1. Quando solicitado, insira **2** para transcrever mensagens.
1. Veja a transcrição de cada mensagem.

    Cada arquivo é reproduzido automaticamente, para que você possa ouvir a mensagem.

## Limpar

Se você terminou de explorar o Azure Speech in Foundry Tools, deve excluir os recursos que criou neste exercício para evitar custos desnecessários no Azure.

1. Abra o [portal do Azure](https://portal.azure.com) e visualize o conteúdo do grupo de recursos onde você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.

## Mais informações

Para obter mais informações sobre o uso das APIs **Speech-to-text** e **Text-to-speech**, consulte a [documentação do Speech-to-text](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e a [documentação do Text-to-speech](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
