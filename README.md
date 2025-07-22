# Explorando a IA Generativa em um Pipeline de ETL com Python

Este guia demonstra como construir um pipeline de Extração, Transformação e Carga (ETL) utilizando a IA Generativa do ChatGPT para a transformação dos dados. O pipeline utilizará pandas para a extração, a API do ChatGPT para a transformação, e uma API REST para o carregamento dos dados.

## Pré-requisitos

*   **Python 3.8+:** Certifique-se de ter uma versão do Python igual ou superior a 3.8 instalada.
*   **Poetry (Opcional):** Recomendado para gerenciamento de dependências. Se não o tiver, pode usar `pip`.
*   **Conhecimento básico de Python:** Familiaridade com a sintaxe e conceitos básicos da linguagem.
*   **Chave da API do ChatGPT:** Você precisará de uma chave de API válida da OpenAI para usar o ChatGPT.
*   **API REST de Destino:** Você precisará ter uma API REST disponível para carregar os dados transformados. Certifique-se de ter a URL e as informações de autenticação necessárias.
*   **Arquivo de Dados de Origem:**  Um arquivo CSV (ou outro formato legível pelo Pandas) para servir como fonte de dados.

## Passo a Passo

### 1. Configuração do Projeto

1.  **Criar o diretório do projeto:**

    ```bash
    mkdir etl-chatgpt-example
    cd etl-chatgpt-example
    ```

2.  **Inicializar o projeto (com Poetry):**

    ```bash
    poetry init --name etl-chatgpt-example --description "ETL Pipeline com ChatGPT" --author "Seu Nome <seu.email@exemplo.com>" --python "^3.8"
    ```
    Preencha as informações solicitadas.

    **Ou (sem Poetry):**
    Crie um arquivo `requirements.txt` para listar as dependências posteriormente.

3.  **Criar o ambiente virtual (se não estiver usando Poetry):**

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

4.  **Estrutura de diretórios:**

    ```
    etl-chatgpt-example/
    ├── etl/
    │   ├── __init__.py
    │   ├── extract.py  # Extração de dados
    │   ├── transform.py # Transformação de dados com ChatGPT
    │   └── load.py     # Carregamento de dados
    ├── data/          # Arquivo de dados de origem
    │   └── source_data.csv
    ├── poetry.toml     # (Se usar Poetry)
    └── README.md       # Este arquivo
    ```

### 2. Instalar as Dependências

1.  **Instalar as dependências (com Poetry):**

    ```bash
    poetry add pandas openai requests python-dotenv
    ```

    **Ou (sem Poetry):**

    Adicione ao `requirements.txt`:

    ```
    pandas
    openai
    requests
    python-dotenv
    ```

    E instale:

    ```bash
    pip install -r requirements.txt
    ```

    **Explicação das dependências:**
    *   `pandas`: Para leitura e manipulação de dados.
    *   `openai`: Para interagir com a API do ChatGPT.
    *   `requests`: Para fazer requisições HTTP para a API REST de destino.
    *   `python-dotenv`: Para carregar variáveis de ambiente de um arquivo `.env`.

### 3. Configurar Variáveis de Ambiente

1.  **Criar o arquivo `.env` na raiz do projeto:**

    ```
    OPENAI_API_KEY=YOUR_OPENAI_API_KEY
    REST_API_URL=YOUR_REST_API_URL
    REST_API_AUTH_TOKEN=YOUR_REST_API_AUTH_TOKEN # Se a API REST precisar de autenticação
    ```

    Substitua `YOUR_OPENAI_API_KEY`, `YOUR_REST_API_URL` e `YOUR_REST_API_AUTH_TOKEN` pelos seus valores reais. **Mantenha este arquivo seguro e não o compartilhe em repositórios públicos!**

2.  **Carregar as variáveis de ambiente:**

    Adicione o seguinte código ao início de cada arquivo Python que precisar acessar as variáveis de ambiente (ou crie um arquivo `config.py` para centralizar isso):

    ```python
    from dotenv import load_dotenv
    import os

    load_dotenv()

    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    REST_API_URL = os.getenv("REST_API_URL")
    REST_API_AUTH_TOKEN = os.getenv("REST_API_AUTH_TOKEN")
    ```

### 4. Implementar a Extração de Dados (Extract)

1.  **Criar o arquivo `etl/extract.py` com o seguinte conteúdo:**

    ```python
    import pandas as pd

    def extract_data(file_path):
        """Extrai dados de um arquivo CSV usando pandas."""
        try:
            df = pd.read_csv(file_path)
            return df
        except FileNotFoundError:
            print(f"Erro: Arquivo não encontrado: {file_path}")
            return None
        except Exception as e:
            print(f"Erro ao ler o arquivo CSV: {e}")
            return None


    if __name__ == '__main__':
        # Exemplo de uso (crie um arquivo 'data/source_data.csv' para testar)
        df = extract_data("data/source_data.csv")
        if df is not None:
            print(df.head())
    ```

    **Explicação:**
    *   Importa a biblioteca `pandas`.
    *   Define a função `extract_data` que recebe o caminho do arquivo CSV como argumento.
    *   Utiliza `pd.read_csv` para ler o arquivo e retornar um DataFrame do pandas.
    *   Trata exceções para lidar com arquivos não encontrados e erros de leitura.
    *   Um bloco `if __name__ == '__main__':` com um exemplo de uso (para testes).

### 5. Implementar a Transformação de Dados com ChatGPT (Transform)

1.  **Criar o arquivo `etl/transform.py` com o seguinte conteúdo:**

    ```python
    import openai
    import os
    from dotenv import load_dotenv

    load_dotenv()

    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    openai.api_key = OPENAI_API_KEY


    def transform_data(df, prompt_instruction):
        """Transforma os dados utilizando o ChatGPT."""
        transformed_data = []
        for index, row in df.iterrows():
            try:
                # Construir o prompt para o ChatGPT com base na linha do DataFrame
                prompt = f"{prompt_instruction}\nData: {row.to_dict()}"

                response = openai.Completion.create(
                    engine="text-davinci-003", # Escolha o modelo adequado
                    prompt=prompt,
                    max_tokens=150,  # Ajuste conforme necessário
                    n=1,           # Número de respostas a gerar
                    stop=None,       # Critério de parada (opcional)
                    temperature=0.7, # Controla a aleatoriedade da resposta
                )

                transformed_text = response.choices[0].text.strip()
                transformed_data.append(transformed_text)

            except Exception as e:
                print(f"Erro ao transformar a linha {index}: {e}")
                transformed_data.append(None)  # ou algum valor padrão

        # Adicionar os dados transformados ao DataFrame
        df['transformed_data'] = transformed_data
        return df


    if __name__ == '__main__':
        # Exemplo de uso (requer um DataFrame e um prompt)
        import pandas as pd
        data = {'nome': ['João', 'Maria'], 'idade': [30, 25]}
        df = pd.DataFrame(data)

        prompt = "Resuma as informações da seguinte pessoa em uma frase:"
        transformed_df = transform_data(df, prompt)
        print(transformed_df)
    ```

    **Explicação:**
    *   Importa a biblioteca `openai`.
    *   Carrega a chave da API do ChatGPT da variável de ambiente `OPENAI_API_KEY`.
    *   Define a função `transform_data` que recebe um DataFrame e um `prompt_instruction` (instrução para o ChatGPT) como argumentos.
    *   Itera sobre as linhas do DataFrame.
    *   Para cada linha, constrói um prompt para o ChatGPT, incluindo a `prompt_instruction` e os dados da linha.
    *   Chama a API do ChatGPT (`openai.Completion.create`) para gerar uma resposta.
    *   Extrai o texto da resposta e o adiciona a uma lista `transformed_data`.
    *   Adiciona uma nova coluna "transformed_data" ao DataFrame com os dados transformados.
    *   Trata exceções para lidar com erros na chamada da API.
    *   Um bloco `if __name__ == '__main__':` com um exemplo de uso.

    **Importante:** A escolha do `engine` (modelo) do ChatGPT e os parâmetros como `max_tokens` e `temperature` dependem do seu caso de uso e podem precisar ser ajustados para obter os melhores resultados.  O modelo `text-davinci-003` é um modelo poderoso, mas pode ser caro. Considere usar modelos mais recentes e/ou mais baratos se possível.

    **Segurança:** Lembre-se de monitorar o uso da sua chave de API do ChatGPT para evitar custos inesperados e proteger sua conta.

### 6. Implementar o Carregamento de Dados (Load)

1.  **Criar o arquivo `etl/load.py` com o seguinte conteúdo:**

    ```python
    import requests
    import json
    import os
    from dotenv import load_dotenv

    load_dotenv()

    REST_API_URL = os.getenv("REST_API_URL")
    REST_API_AUTH_TOKEN = os.getenv("REST_API_AUTH_TOKEN")

    def load_data(data, api_url, auth_token=None):
        """Carrega os dados para uma API REST."""
        headers = {'Content-type': 'application/json'}
        if auth_token:
            headers['Authorization'] = f'Bearer {auth_token}'  # Ajuste conforme o tipo de autenticação

        try:
            response = requests.post(api_url, data=json.dumps(data), headers=headers)
            response.raise_for_status()  # Levanta uma exceção para status code de erro

            print(f"Dados carregados com sucesso. Status Code: {response.status_code}")
            return response.json()  # Retorna a resposta da API, se houver

        except requests.exceptions.RequestException as e:
            print(f"Erro ao carregar os dados: {e}")
            return None


    if __name__ == '__main__':
        # Exemplo de uso (requer a URL da API REST e os dados)
        data = {'chave1': 'valor1', 'chave2': 'valor2'}
        api_response = load_data(data, REST_API_URL, REST_API_AUTH_TOKEN)

        if api_response:
            print("Resposta da API:", api_response)
    ```

    **Explicação:**
    *   Importa a biblioteca `requests`.
    *   Carrega a URL da API REST de destino da variável de ambiente `REST_API_URL`.
    *   Define a função `load_data` que recebe os dados, a URL da API e um token de autenticação (opcional) como argumentos.
    *   Define os cabeçalhos da requisição, incluindo o tipo de conteúdo como JSON e, se fornecido, o token de autenticação.  Adapte o tipo de autenticação se a API REST usar um método diferente (por exemplo, API Key).
    *   Utiliza `requests.post` para enviar os dados para a API.
    *   Trata exceções para lidar com erros de conexão e status codes de erro.
    *   Imprime o status code da resposta e, se houver, a resposta da API.
    *   Um bloco `if __name__ == '__main__':` com um exemplo de uso.

### 7. Orchestrar o Pipeline de ETL

1.  **Criar um arquivo principal (por exemplo, `main.py` na raiz do projeto) para orquestrar o pipeline:**

    ```python
    from etl.extract import extract_data
    from etl.transform import transform_data
    from etl.load import load_data
    import pandas as pd

    # 1. Extração
    data_path = "data/source_data.csv"  # Substitua pelo caminho real do seu arquivo
    df = extract_data(data_path)

    if df is None:
        print("Falha na extração. Abortando o pipeline.")
        exit()

    # 2. Transformação
    prompt_instruction = "Traduza o texto para o inglês:" # Personalize a instrução
    transformed_df = transform_data(df, prompt_instruction)

    if transformed_df is None:
        print("Falha na transformação. Abortando o pipeline.")
        exit()

    # 3. Carregamento
    # Converter o DataFrame transformado para o formato que a API REST espera (dicionário, JSON, etc.)
    # Aqui, vamos converter cada linha do DataFrame para um dicionário e carregar individualmente
    for index, row in transformed_df.iterrows():
        data_to_load = row.to_dict() # Ou use transformed_df.to_dict(orient='records') para obter uma lista de dicionários

        api_response = load_data(data_to_load, REST_API_URL, REST_API_AUTH_TOKEN)

        if api_response is None:
            print(f"Falha ao carregar os dados da linha {index}.")


    print("Pipeline ETL concluído.")
    ```

    **Explicação:**
    *   Importa as funções `extract_data`, `transform_data` e `load_data` dos respectivos módulos.
    *   Define o caminho para o arquivo de dados de origem.
    *   Chama `extract_data` para extrair os dados para um DataFrame.
    *   Define a instrução para o ChatGPT.
    *   Chama `transform_data` para transformar os dados utilizando o ChatGPT.
    *   Converte o DataFrame transformado (ou cada linha) para o formato esperado pela API REST.
    *   Chama `load_data` para carregar os dados transformados para a API REST.
    *   Trata erros em cada etapa do pipeline.
    *   Imprime uma mensagem de conclusão.

### 8. Executar o Pipeline

1.  **Executar o arquivo `main.py`:**

    ```bash
    python main.py
    ```

    O pipeline será executado, extraindo os dados do arquivo CSV, transformando-os utilizando o ChatGPT, e carregando-os para a API REST de destino.

## Próximos Passos e Considerações

*   **Tratamento de Erros:** Implementar tratamento de erros mais robusto em cada etapa do pipeline.
*   **Logging:** Adicionar logging para rastrear a execução do pipeline e diagnosticar problemas.
*   **Monitoramento:** Implementar monitoramento para acompanhar o desempenho do pipeline e detectar anomalias.
*   **Agendamento:** Agendar a execução do pipeline para que ele seja executado automaticamente em intervalos regulares.
*   **Qualidade dos Dados:** Implementar verificações de qualidade dos dados para garantir que os dados transformados sejam precisos e consistentes.
*   **Customização do Prompt:** Experimentar diferentes prompts para o ChatGPT para obter os melhores resultados na transformação dos dados.  A qualidade do prompt é crucial para o sucesso da transformação.
*   **Modelos do ChatGPT:** Explore diferentes modelos do ChatGPT e ajuste os parâmetros da API para otimizar o desempenho e o custo.
*   **Escalabilidade:** Considere usar frameworks como Dask ou Spark para processar grandes volumes de dados.
*   **Segurança:** Armazene as chaves de API e outras informações confidenciais de forma segura, utilizando gerenciadores de segredos ou outras técnicas de segurança.

Este guia fornece um ponto de partida para explorar a IA Generativa do ChatGPT em um pipeline de ETL. Adapte o código e as configurações para atender às suas necessidades específicas. Lembre-se que a integração com o ChatGPT depende muito da qualidade do seu prompt e da natureza dos seus dados.  Experimente e itere para obter os melhores resultados.