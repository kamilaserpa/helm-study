# HELM

Iniciando minikube:
`minikube start --memory=6G --cpus=4`

Projeto inicial do curso: https://github.com/alura-cursos/Kubernetes-helm/tree/projeto_inicial

## O que é o Helm?

Helm é um gerenciador de pacotes para Kubernetes, tamb;em conhecidos como charts. Ele facilita a definição, instalação e atualização de aplicações Kubernetes.

### Funções do Helm

 - Chart: é um pacote do Kubernetes que contém todos os recursos de configuração necessários para executar uma aplicação, ferramenta ou serviço dentro de um cluster Kubernetes.
 - Repository: um local onde os charts são armazenados e podem ser pesquisados, baixados e instalados.
 - Release: uma instância de um chart implementado em um cluster Kubernetes.

### Vantagens do Helm
1. Gestão Simplificada: facilita a gestão de aplicações Kubernetes, tornando mais fácil instalar, atualizar e gerenciar aplicações complexas.
2. Reutilização: charts podem ser reutilizados e compartilhados, economizando tempo e esforço na configuração de novas aplicações.
3. Versão: permite a versão de aplicações, facilitando a reversão para versões anteriores em caso de problemas.
4. Automação: automatiza o processo de deploy, reduzindo o risco de erro humano.
5. Comunidade: ampla comunidade de usuários e contribuidores, oferecendo muitos charts prontos para uso.
6. Consistência: carante que a mesma configuração é aplicada em diferentes ambientes, garantindo consistência.

### Desvantagens do Helm
1. Complexidade Inicial: pode ser complexo para iniciantes que ainda estão aprendendo Kubernetes.
2. Manutenção de Charts: manter e atualizar charts personalizados pode ser trabalhoso.
3. Curva de Aprendizado: pode haver uma curva de aprendizado significativa, especialmente para desenvolvedores que não estão familiarizados com Kubernetes.
4. Dependências: charts podem ter muitas dependências, o que pode complicar a gestão e a resolução de problemas.

### Funções específicas do Helm

 - helm install: instala um chart.
 - helm upgrade: atualiza uma release com uma nova versão do chart.
 - helm rollback: reverte uma release para uma versão anterior.
 - helm list: lista todas as releases instaladas.
 - helm search: pesquisa por charts em repositórios.
 - helm repo add: adiciona um novo repositório de charts.
 - helm uninstall: remove uma release do cluster.

### Exemplo de instalação de um chart

Podemos pesquisar por pacotes em https://artifacthub.io/.
Vamos instalar o MySQL, por exemplo, em ArtifactHub buscamos MySQL, clicamos em "Install", serão exibidos os comandos para a instalação. Com o Docker e minikube em execução:
```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-mysql bitnami/mysql --version 14.0.3
```

Serão exibidos vários dados, dentre eles o user e senha do banco de dados.
```bash
  Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default my-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
# Exemplo de como capturar a senha do MySQL
$ kubectl get secret --namespace default my-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d
IaZhfk8lSF
```

## Chart
Um chart no Helm é um pacote de configuração que contém todos os arquivos necessários para descrever os recursos Kubernetes.

###Estrutura de Diretórios de um Chart
Um chart é organizado em um conjunto específico de diretórios e arquivos. Aqui está um exemplo típico da estrutura de diretórios de um chart:

```
my-chart/
├── Chart.yaml             # Contém informações sobre o chart
├── values.yaml            # Valores padrão de configuração do chart
├── charts/                # Contém charts dependentes
├── templates/             # Arquivos de template que definem os recursos Kubernetes
├── templates/_helpers.tpl # Arquivo de ajuda para templates
├── .helmignore            # Arquivo para listar arquivos/padrões a serem ignorados
```

### Descrição dos Arquivos e Diretórios
 - Chart.yaml: Contém metadados sobre o chart, como nome, versão e descrição
 - values.yaml: Contém valores de configuração padrão para o chart. Esses valores podem ser substituídos durante a instalação ou atualização do chart.
 - charts/: Contém charts dependentes, se houver. Esses são charts que são instalados junto com o chart principal.
 - templates/: Contém arquivos de template que são usados para gerar os manifestos Kubernetes. Exemplos de arquivos de template incluem deployment.yaml, service.yaml, etc.
 ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: {{ .Values.configMap.name }}
    data:
    {{- range $key, $value := .Values.configMap.data}} # faz um loop capturando o key e value declarados no arquivo helm/value.yaml em configMap.data
    {{$key}}: "{{$value}}" # declara como usar essas variáveis após capturadas
    {{- end}}
  ```
 - templates/_helpers.tpl: Contém definições de funções de template que podem ser usadas em outros arquivos de template. Essas funções ajudam a manter os templates DRY (Don't Repeat Yourself).
 - .helmignore: Funciona de forma semelhante a um .gitignore, especificando os arquivos e diretórios que devem ser ignorados ao empacotar o chart.

#### tempaltes/_helpers.tpl

O arquivo [tempaltes/_helpers.tpl](/helm/templates/_helpers.tpl) é útil para definir funções auxiliares reutilizáveis que podem ser chamadas em qualquer template do seu chart.

A propriedade `{{.Release.Name}}` não é definida em nenhum arquivo do seu projeto. Ela é uma variável built-in do Helm que é automaticamente disponibilizada durante a renderização dos templates.

Quando instalar o chart, por exemplo:
```bash
 helm install alura-foods-prod ./helm
```

O {{.Release.Name}} será substituído por "alura-foods-prod".
Para usar em um template:
```bash
# _helpers.tpl
{{- define "alura-foods-app.labels"-}}
app.kubernetes.io/name: {{.Chart.Name}}
app.kubernetes.io/instance: {{.Release.Name}}
app.kubernetes.io/version: {{.Chart.AppVersion}}
app.kubernetes.io/managed-by: {{.Release.Service}}
{{- end}}
# anothe template
metadata:
  labels:
    {{- include "alura-foods-app.labels" . | nindent 4 }} 
    # nindent 4 significa que adiciona 4 espaços iniciais pra alinhar o yaml
```

### Criando o nosso próprio Helm Chart
