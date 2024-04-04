# Servidor de Horas Simples (simple-time-api)

## O Servidor

A aplicação flask é uma simples API que serve as horas de acordo com o horário do servidor no formato json:

```python
from flask import Flask, jsonify
from datetime import datetime

app = Flask(__name__)

@app.route('/')
def get_current_time():
    current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    return jsonify({'time': current_time})

if __name__ == '__main__':
    app.run(debug=True,host='0.0.0.0', port=5000)
```

## Criando a Imagem

O Dockerfile abaixo:

**1** - Define qual imagem Linux eu quero utilizar, nesse caso, uma imagem especifica para trabalhar com python.

**2** - Define um diretório para por os arquivos da aplicação

**3** - Copia os conteúdos do diretório atual para o diretório especificado. **Nesse caso** o mesmo que o da linha 2, mas poderia ser um terceiro diretório.

**4** - Instala as dependências utilizadas pelo `app.py`

**5** - Expôe uma porta, nesse caso, a utiliza pela aplicação flask:

**6** - Comando que coloca a aplicação flask para funcionar.

```bash
1 FROM python:3-alpine3.15
2 WORKDIR /time-server
3 COPY . /time-server
4 RUN pip install -r requirements.txt
5 EXPOSE 5000
6 CMD python ./app.py
```

Para construir a imagem a partir do Dockerfile:

```bash
$ docker build --tag clusterminator/time-server:2.0 .
```

## Docker

Para executar o servidor apenas com o docker, basta usar:

```bash
$ docker run -d -p 80:5000 clusterminator/time-server:2.0
```

Ao acessar o endereço [localhost](http://localhost) você terá uma resposta similar à:

```json
{
  "time": "2024-04-04 23:14:18"
}
```

## Kubernetes

Para executar a aplicação usando kubernetes, acesse o diretório dos containers:

```bash
$ cd kubernetes-yaml-files
```

Depois execute os comandos abaixo:

```bash
$ kubectl apply -f pod-time-server.yaml
$ kubectl apply -f NodePort-time-server.yaml
```

O primeiro vai inicializar o servidor em si e o segundo vai criar o serviço de NodePort, expondo o servidor na porta 30000 do seu nó (isso é, sua máquina) a aplicação vai estar disponível em [localhost:30000](http://localhost:30000).
