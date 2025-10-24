#  Projeto GitOps: Online Boutique com Kubernetes e ArgoCD 🚀

Este projeto implementa a aplicação de microsserviços "Online Boutique" (um site de e-commerce de demonstração 🛍️) em um cluster Kubernetes local (Rancher Desktop). O deploy é gerenciado inteiramente através de práticas de GitOps, usando o ArgoCD como ferramenta de entrega contínua.

O Git é usado como a única fonte para o estado desejado da aplicação. O ArgoCD monitora o repositório Git e aplica automaticamente quaisquer alterações ao cluster Kubernetes, tornando o processo de deploy mais rápido, seguro e auditável.

## 1. Objetivo do Projeto 🎯

Executar o conjunto de microsserviços "Online Boutique" em um cluster Kubernetes local usando Rancher Desktop, com o deploy sendo controlado por GitOps através do ArgoCD, a partir de um repositório público no GitHub.

## 2. Arquitetura 🏗️

* **Cluster Kubernetes:** [Rancher Desktop](https://rancherdesktop.io/) 🖥️
* **Aplicação:** [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/release/kubernetes-manifests.yaml) (Demo de Microsserviços do Google) 🛒
* **Ferramenta de GitOps:** ArgoCD ⛵
* **Fonte:** [Repositório Git](https://github.com/GoogleCloudPlatform/microservices-demo) (GitHub) 📂

## 3. Pré-requisitos ✅

Antes de começar, garanta que você possui os seguintes softwares instalados e configurados:

* [ ] **Rancher Desktop** instalado (com Kubernetes habilitado)
* [ ] **Kubectl** configurado (verifique com `kubectl get nodes`)
* [ ] **Git** instalado (verifique com `git --version`)
* [ ] **Conta no GitHub** para criar um repositório público

## 4. Tutorial Passo a Passo 🗺️

Siga estas etapas para configurar o ambiente e implantar a aplicação.

### Etapa 1: Preparar o Repositório Git 📂

O ArgoCD precisa de um repositório para monitorar.Vamos criar um repositório limpo contendo apenas os manifestos YAML da aplicação.

1.  **Criar o Repositório:** No GitHub, crie um novo repositório público (ex: `gitops-microservices`).
2.  **Clonar o Repositório:** Clone o repositório vazio para sua máquina local.
    ```bash
    git clone [https://github.com/](https://github.com/)<SEU-USUARIO>/gitops-microservices.git
    cd gitops-microservices
    ```
3.  **Criar a Estrutura de Pastas:** Crie a estrutura de diretórios recomendada.
    ```bash
    mkdir k8s
    ```
4.  **Adicionar o Manifesto:** Baixe o arquivo `kubernetes-manifests.yaml` do projeto original e salve-o dentro da pasta `k8s` com o nome `online-boutique.yaml`.
    ```bash
    # Este comando baixa o arquivo e o salva no local correto
    curl -L -o k8s/online-boutique.yaml [https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml](https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml)
    ```
5.  **Fazer Commit e Push:** Envie os novos arquivos para o seu repositório no GitHub.
    ```bash
    git add .
    git commit -m "Adiciona manifestos da Online Boutique"
    git push origin main
    ```

### Etapa 2: Instalar o ArgoCD no Cluster ⛵

Agora, vamos instalar o ArgoCD no seu cluster Kubernetes (Rancher Desktop).

1.  **Criar o Namespace:** O `namespace` é um espaço virtual isolado para o ArgoCD.
    ```bash
    kubectl create namespace argocd
    ```
    *(**Nota:** Um `namespace` não é um diretório no seu PC, mas sim uma divisão lógica dentro do cluster Kubernetes.)*

2.  **Aplicar o Manifesto de Instalação:**
    ```bash
    kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
    ```

3.  **Verificar Instalação:** Espere até que todos os pods do ArgoCD estejam com status `Running`.
    ```bash
    # O -w (watch) assiste em tempo real. Pressione Ctrl+C para sair.
    kubectl get pods -n argocd -w
    ```

### Etapa 3: Acessar a Interface Web do ArgoCD 🌐

Vamos acessar o dashboard do ArgoCD para gerenciar a aplicação.

1.  **Fazer Port-Forward:** Crie um túnel seguro para o serviço do ArgoCD. Abra um **novo terminal** e execute:
    ```bash
    # Conecta sua porta local 8080 à porta 443 do serviço argocd-server
    kubectl port-forward service/argocd-server -n argocd 8080:443
    ```
    *(Este terminal ficará "travado" mantendo a conexão.)*

2.  **Acessar a UI:** Abra seu navegador e acesse `https://localhost:8080`.
    *(Ignore o aviso de segurança ⚠️; o certificado é autoassinado.)*

3.  **Obter a Senha:** O usuário é `admin`. A senha inicial é gerada automaticamente. Abra um **terceiro terminal** para obtê-la.
    *(Este comando é para PowerShell, o padrão no Windows Terminal)*
    ```powershell
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
    ```
    *(Copie a senha 🔑 e cole no navegador para fazer login.)*
    <img width="1920" height="760" alt="login_argocd" src="https://github.com/user-attachments/assets/d6a4105b-92ee-445e-9728-5aff1080b640" />


### Etapa 4: Criar e Sincronizar a Aplicação no ArgoCD 🔄

Vamos dizer ao ArgoCD para monitorar seu repositório Git.

1.  Na interface do ArgoCD, clique em **"+ NEW APP"**.
2.  Preencha os campos:
    * **Application Name:** `online-boutique`
    * **Project Name:** `default`
    * **Sync Policy:** `Manual`
    * **Repository URL:** A URL do seu repositório (ex: `https://github.com/SEU-USUARIO/gitops-microservices.git`)
    * **Path:** `k8s` (a pasta que criamos)
    * **Cluster URL:** `https://kubernetes.default.svc` (o cluster local)
    * **Namespace:** `default` (onde a aplicação será instalada)
    <img width="1920" height="827" alt="config argo 1" src="https://github.com/user-attachments/assets/116b0ece-334e-4c44-bf3d-76ee4d55260d" />
    <img width="1920" height="864" alt="config argo 2" src="https://github.com/user-attachments/assets/23ebaf30-c4e4-4e05-84cf-bb629fefe952" />

3.  Clique em **"CREATE"**.
4.  Clique no card da aplicação e, em seguida, clique em **"SYNC"** para iniciar o deploy.

### Etapa 5: Acessar a Aplicação Online Boutique 🛍️

Após a sincronização, os pods da aplicação serão iniciados.

1.  **Fazer Port-Forward para a Aplicação:** A aplicação é exposta por um serviço chamado `frontend`. O acesso ao ArgoCD (Etapa 3) já está usando a porta `8080`, então vamos usar a `8081` para evitar conflito.
    * Abra um **novo terminal** e execute:
    ```bash
    # Conecta sua porta local 8081 à porta 80 do serviço frontend
    kubectl port-forward service/frontend 8081:80
    ```
2.  **Acessar o Site:**
    * Abra seu navegador e acesse: `http://localhost:8081`
    * Você deverá ver o site da Online Boutique! 🎉

