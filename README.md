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

Você vai encontrar dois arquivos:

### pod-time-server.yaml 
```yaml
 1 apiVersion: v1
 2 kind: Pod
 3 metadata:
 4   name: pod-time-server
 5   labels:
 6     app: pod-time-server
 7 spec:
 8   containers:
 9   - name: time-server-container
10     image: clusterminator/time-server:2.0
11     ports:
12     - containerPort: 5000
```

1. `apiVersion`: Especifica a versão da API Kubernetes que está sendo usada, nesse caso, a versão 1.

2. `kind`: Especifica o tipo de objeto que queremos criar, nesse caso, um Pod.

3. `metadados`: Contém metadados sobre o pod, como nome e rótulos.

4. `name`: Especifica o nome do Pod como `pod-time-server`.

5. `labels`: Campo para especificar rótulos que serão usados para selecionar o pod.

6. `app`: Atribui o rótulo `pod-time-server` ao pod. Este rótulo pode ser usado por outros pod e serviços para identificar e selecionar pods com este rótulo.

7. `spec`: Define a especificação do Pod, incluindo seus contêineres e outras configurações.

8. `containers`: Especifica uma lista de contêineres a serem executados dentro do Pod.

9. `name`: Define o nome do contêiner.

10. `image: clusterminator/time-server:2.0`: Especifica a imagem Docker a ser usada para o contêiner.

11. `ports`: Especifica uma lista de portas a serem expostas no contêiner.

12. `containerPort`: Indica que o contêiner escuta na porta `5000`. Essa porta será exposta e poderá ser acessada de dentro do pod e, se configurada, também de fora do pod.

### NodePort-time-server.yaml
```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: NodePort-time-server
  spec:
1   type: NodePort
2   selector:
3     app: time-server-pod
4   ports:
5     - port: 80
6       targetPort: 5000
7       nodePort: 30000
```
1. `type:`: Especifica o tipo de serviço como `NodePort`. Este tipo expõe o Serviço no IP de cada Node em uma porta estática (o `nodePort`).

2. `selector`: Especifica os pods para os quais este serviço encaminhará o tráfego. Nesse caso, ele seleciona Pods com o rótulo `app: time-server-pod`.

3. `ports`: Especifica uma lista de portas a serem expostas pelo Serviço.

4. `port`: Indica que o Serviço irá escutar na porta `80`. Esta é a porta à qual os clientes se conectarão ao acessar o Serviço.

5. `targetPort`: Especifica a porta nos pods para a qual o serviço encaminhará o tráfego. Nesse caso, é a porta `5000`, que é a porta em que os Pods estão escutando.

6. `nodePort`: Especifica o número da porta estática em cada nó onde o serviço será exposto. Esta é uma porta com numeração alta na faixa `30000-32767` que clientes externos podem usar para acessar o Serviço. Neste caso, é `30000`.

### Iniciando o cluster

Depois execute os comandos abaixo:

```bash
$ kubectl apply -f pod-time-server.yaml
$ kubectl apply -f NodePort-time-server.yaml
```

O primeiro vai inicializar o servidor em si e o segundo vai criar o serviço de NodePort, expondo o servidor na porta 30000 do seu host (isso é, sua máquina) a aplicação vai estar disponível em [localhost:30000](http://localhost:30000).
