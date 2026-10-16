---
lab:
    title: 'Traduzir texto e fala'
    description: Implementar tradução com Azure Translator e Azure Speech no Foundry Tools.
    duration: 30
    level: 300
    islab: true
    status: 'released'
layout: default
---

# Traduzir texto e fala

**Azure Translator in Foundry Tools** é um serviço que permite traduzir texto entre idiomas. Da mesma forma, **Azure Speech in Foundry Tools** fornece serviços de tradução para fala. Neste exercício, você usará esses serviços para criar aplicativos de tradução que traduzem a entrada em qualquer idioma compatível para o idioma de destino de sua escolha.

Embora este exercício seja baseado em Python, você pode desenvolver aplicativos de tradução de texto usando vários SDKs específicos de linguagem, incluindo:

- [Biblioteca de cliente do Azure Translator para Python](https://pypi.org/project/azure-ai-translation-text/)
- [Biblioteca de cliente do Azure Translator para .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Biblioteca de cliente do Azure Translator para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)
- [Azure AI Speech SDK para Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [Azure AI Speech SDK para .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [Azure AI Speech SDK para JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Este exercício leva aproximadamente **30** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de começar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account) ativa
- [Visual Studio Code](https://code.visualstudio.com/) instalado
- [Python versão **3.13.xx**](https://www.python.org/downloads/release/python-31312/) instalado\*
- [Git](https://git-scm.com/install/) instalado e configurado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) instalado

> \* O Python 3.14 está disponível, mas algumas dependências ainda não foram compiladas para essa versão. O laboratório foi testado com sucesso com o Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [portal do Microsoft Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche quaisquer dicas ou painéis de início rápido que sejam abertos na primeira vez que você entrar e, se necessário, use o logotipo do Foundry no canto superior esquerdo para navegar até a página inicial.

1. Se ainda não estiver habilitada, na barra de ferramentas na parte superior da página, habilite a opção **New Foundry**. Em seguida, se solicitado, crie um novo projeto com um nome exclusivo; expandindo a área **Advanced options** para especificar as seguintes configurações para seu projeto:
    - **Foundry resource**: *Use o nome padrão para seu recurso (geralmente {project_name}-resource)*\*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Region**: Selecione qualquer região disponível

    > **Observação**: Use uma região recomendada do Microsoft Foundry. A disponibilidade de modelos pode variar por região.

    > **Dica**: \* Lembre-se do nome do Foundry resource — você precisará dele mais tarde!

1. Aguarde a criação do projeto. Em seguida, exiba a página inicial do seu projeto.

## Explorar o Azure Translator no Foundry Tools no portal

Você pode usar o playground do Azure Translator no portal Foundry para experimentar o serviço.

1. Agora você está pronto para **Start building**. Selecione **Explore playgrounds** (ou, na página **Build**, selecione a guia **Models**) para ver os modelos no seu projeto.
1. Na página **Models**, selecione a guia **AI services** para ver a lista de serviços do Azure no Foundry Tools.
1. Na lista de ferramentas, selecione **Azure Translator - Text translation**.
1. No playground do Text translator, na área **Source text**, insira o texto `Hello world!`. Em seguida, na área **Translation**, selecione qualquer idioma e use o botão **Translate** para gerar a tradução.
1. Experimente mais alguns idiomas.
1. Selecione a guia **Code** para ver um código de exemplo para usar o Azure Translator; e observe a variável **ENDPOINT** usada no código para a REST API, que deve ser semelhante a `https://{foundry-resource-name}.cognitiveservices.azure.com/`.

    Esse endpoint usa um formato mais antigo para Azure AI Services, mas ainda é usado para se conectar ao recurso do Azure Translator em um Foundry resource. Você também pode usá-lo para se conectar às ferramentas do Azure Speech.

    > **Dica**: Você vai precisar do endpoint mais tarde!

## Obter arquivos do aplicativo no GitHub

Os arquivos iniciais do aplicativo necessários para desenvolver o aplicativo de tradução são fornecidos em um repositório do GitHub.

1. Abra o Visual Studio Code.
1. Abra a paleta de comandos (Ctrl+Shift+P) e use o comando `Git:clone` para clonar o repositório `https://github.com/microsoftlearning/mslearn-ai-language` para uma pasta local (não importa qual). Em seguida, abra-o.

    Você pode ser solicitado a confirmar que confia nos autores.

1. No Visual Studio Code, exiba o painel **Extensions**; e, se ainda não estiver instalada, instale a extensão **Python**.
1. Na **Command Palette**, use o comando `python:select interpreter`. Em seguida, selecione um ambiente existente, se tiver um, ou crie um novo ambiente **Venv** com base na sua instalação do Python 3.1x.

    > **Dica**: Se você for solicitado a instalar dependências, pode instalar as do arquivo *requirements.txt* na pasta */Labfiles/07-translation/Python/translators*; mas tudo bem se não fizer isso agora — vamos instalá-las depois!

    > **Dica**: Se preferir usar o terminal, você pode criar seu ambiente **Venv** com `python -m venv labenv` e depois ativá-lo com `\labenv\Scripts\activate`.

## Criar um aplicativo de tradução de texto

Agora você está pronto para usar o Azure Translator para implementar a tradução de texto.

1. Após a clonagem do repositório, no painel Explorer, navegue até a pasta que contém os arquivos de código do aplicativo em **/Labfiles/07-translation/Python/translators**. Os arquivos do aplicativo incluem:
    - **.env** (o arquivo de configuração do aplicativo)
    - **requirements.txt** (as dependências de pacotes Python que precisam ser instaladas)
    - **translate-text.py** (o arquivo de código para o aplicativo de texto)
    - **translate-speech.py** (o arquivo de código para o aplicativo de fala)

### Configurar seu aplicativo de tradução de texto

1. No painel **Explorer**, na pasta **translators**, selecione o arquivo **.env** para abri-lo. Em seguida, atualize os valores de configuração para refletir o **endpoint** do Cognitive Services para o seu Foundry resource.

    > **Importante**: O endpoint deve ser *https://{YOUR_FOUNDRY_RESOURCE}.cognitiveservices.azure.com/*. O nome do Foundry Resource geralmente tem o formato *{project_name}-resource*.

    Salve o arquivo de configuração modificado.

1. No painel **Explorer**, clique com o botão direito na pasta **translators** que contém os arquivos do aplicativo e selecione **Open in integrated terminal** (ou abra um terminal no menu **Terminal** e navegue até a pasta */Labfiles/07-translation/Python/translators*.)

    > **Observação**: Abrir o terminal no Visual Studio Code ativará automaticamente o ambiente Python. Talvez seja necessário habilitar a execução de scripts no seu sistema.

1. Verifique se o terminal está aberto na pasta **translators** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Instale o Azure Translator SDK, o Speech SDK e outros pacotes necessários executando o seguinte comando:

    ```
    pip install -r requirements.txt
    ```

### Adicionar código para traduzir texto

1. No painel **Explorer**, na pasta **translators**, abra o arquivo **translate-text.py**.

1. Revise o código existente. Você adicionará código para trabalhar com o Azure Translator.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. No início do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar os namespaces que você precisará para usar o Translator SDK:

    ```python
   # import namespaces
   from azure.identity import DefaultAzureCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. Na função **main**, observe que o código existente lê as configurações de configuração.
1. Localize o comentário **Create client using endpoint and credential** e adicione o seguinte código:

    ```python
   # Create client using endpoint and credential
   credential = DefaultAzureCredential()
   client = TextTranslationClient(credential=credential, endpoint=foundry_endpoint)
    ```

1. Localize o comentário **Choose target language** e adicione o seguinte código, que usa o serviço Text Translator para retornar a lista de idiomas com suporte para tradução e solicita ao usuário que selecione um código de idioma para o idioma de destino:

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Localize o comentário **Translate text** e adicione o seguinte código, que solicita repetidamente ao usuário um texto a ser traduzido, usa o serviço Azure AI Translator para traduzi-lo para o idioma de destino (detectando automaticamente o idioma de origem) e exibe os resultados até que o usuário digite *quit*:

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Salve as alterações no arquivo de código. Em seguida, no painel do terminal, use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Entrar no Azure de forma interativa usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu Foundry resource.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python translate-text.py
    ```

1. Quando solicitado, insira um idioma de destino válido da lista no link exibido.
1. Insira uma frase para ser traduzida (por exemplo, `This is a test` ou `C'est un test`) e veja os resultados, que devem detectar o idioma de origem e traduzir o texto para o idioma de destino.
1. Quando terminar, digite `quit`. Você pode executar o aplicativo novamente e escolher um idioma de destino diferente.

## Criar um aplicativo de tradução de fala

Agora você está pronto para usar o Azure Speech para implementar tradução de texto.

### Configurar seu aplicativo de tradução de fala

1. Na pasta **translators**, verifique se o arquivo .env contém o **endpoint** do seu Foundry resource (o Azure Speech pode usar as mesmas informações que o Azure Translator para se conectar ao seu Foundry resource).
1. Verifique se o terminal está aberto na pasta **translators** com o prefixo **(.venv)** para indicar que o ambiente Python que você criou está ativo.
1. Se você não instalou anteriormente os pacotes necessários, insira o seguinte comando para fazê-lo agora:

    ```
    pip install -r requirements.txt
    ```

### Adicionar código para traduzir fala

1. No painel **Explorer**, na pasta **translators**, abra o arquivo **translate-speech.py**.

1. Revise o código existente. Você adicionará código para trabalhar com o Azure Speech.

    > **Dica**: Ao adicionar código ao arquivo, certifique-se de manter a indentação correta.

1. No início do arquivo de código, abaixo das referências de namespace existentes, localize o comentário **Import namespaces** e adicione o seguinte código para importar o namespace necessário para usar o Speech SDK:

   ```python
   # Import namespaces
   from azure.identity import DefaultAzureCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Na função **main**, sob o comentário **Get configuration settings**, observe que o código carrega o endpoint que você definiu no arquivo de configuração.

1. Localize o seguinte código sob o comentário **Configure translation** e adicione o seguinte código para configurar sua conexão ao endpoint do Foundry para o Azure Speech e preparar a tradução de fala em inglês dos EUA para francês, espanhol e híndi:

    ```python
   # Configure translation
   credential = DefaultAzureCredential()
   translation_cfg = speech_sdk.translation.SpeechTranslationConfig(
            token_credential=credential,
            endpoint=foundry_endpoint
   )
   translation_cfg.speech_recognition_language = 'en-US'
   translation_cfg.add_target_language('fr')
   translation_cfg.add_target_language('es')
   translation_cfg.add_target_language('hi')
   audio_in_cfg = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(
        translation_config=translation_cfg,
        audio_config=audio_in_cfg
   )
   print('Ready to translate from',translation_cfg.speech_recognition_language)
    ```

1. Você usará o **SpeechTranslationConfig** para traduzir fala em texto, mas também usará um **SpeechConfig** para sintetizar traduções em fala. Adicione o seguinte código sob o comentário **Configure speech for synthesis of translations**:

    ```python
   # Configure speech for synthesis of translations
   speech_cfg = speech_sdk.SpeechConfig(
        token_credential=credential, endpoint=foundry_endpoint)
   voices = {
        "fr": "fr-FR-HenriNeural",
        "es": "es-ES-ElviraNeural",
        "hi": "hi-IN-MadhurNeural"
   }
   print('Ready to use speech service.')
    ```

1. Agora é hora de adicionar o código para traduzir a fala do usuário capturada pelo microfone do sistema. Localize o comentário **Translate user speech** e adicione o seguinte código:

    ```python
   # Translate user speech
   print("Speak now...")
   translation_results = translator.recognize_once_async().get()
   print(f"Translating '{translation_results.text}'")
    ```

1. Quando os resultados forem retornados, o aplicativo iterará pelas traduções, imprimindo o texto e reproduzindo a fala sintetizada pelo alto-falante padrão do sistema. Localize o comentário **Print and speak the translation results** e adicione o seguinte código:

    ```python
   # Print and speak the translation results
   translations = translation_results.translations
   for translation_language in translations:

        print(f"{translation_language}: '{translations[translation_language]}'")

        speech_cfg.speech_synthesis_voice_name = voices.get(translation_language)
        audio_out_cfg = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
        speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_cfg, audio_out_cfg)
        speak = speech_synthesizer.speak_text_async(translations[translation_language]).get()
        
        if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
            print(speak.reason)
    ```

1. Salve as alterações no arquivo de código. Em seguida, no painel do terminal, se você ainda não estiver conectado ao Azure (ou sua sessão pode ter expirado), use o seguinte comando para entrar no Azure.

    ```powershell
    az login
    ```

    > **Observação**: Na maioria dos cenários, apenas usar *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Entrar no Azure de forma interativa usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para entrar no Azure. Em seguida, conclua o processo de entrada na linha de comando, visualizando (e confirmando, se necessário) os detalhes da assinatura que contém seu Foundry resource.
1. Depois de entrar, insira o seguinte comando para executar o aplicativo:

    ```
   python translate-speech.py
    ```

1. Quando solicitado, diga algo em voz alta (por exemplo, "*Hello!*").

     O programa deve traduzi-lo para os idiomas especificados no código (francês, espanhol e híndi) e imprimir e falar as traduções.

    > **Observação**: A tradução para híndi pode nem sempre ser exibida corretamente no terminal devido a problemas de codificação de caracteres.

## Limpar recursos

Se você terminou de explorar o Microsoft Foundry, exclua quaisquer recursos que não sejam mais necessários. Isso evita a acumulação de custos desnecessários.

1. Abra o **Azure portal** em [https://portal.azure.com](https://portal.azure.com) e selecione o grupo de recursos que contém os recursos que você criou.
1. Selecione **Delete resource group** e, em seguida, **enter the resource group name** para confirmar. O grupo de recursos será excluído.
