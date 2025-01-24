# Documentação completa 
Documentação no [Confluence](URL DA DOC)

# Template de Integração Continua
Esse repositorio tem o objetivo de possibilitar a integração do codigo a partir de um commit ou PULL REQUEST, para que rode a stack de analise que possibilite uma boa qualidade das aplicações, rodando ferramentas que irão verificar as etapas de entrega, como avaliação de credenciais expostas no repositorio, lint de manifestos, SAST com Sonar e Veracode e publicação das analises.

# Fluxo novo CI/CD
![Diagrama do Fluxo](./PATH/.drawio.svg)
<img src="./PATH/.drawio.svg">


### 1. Branch Feature (`feature/*`)
- **Ação:** Criação de artefato e execução de GitLeaks.
- **Objetivo:** Gerar um artefato entregável para o ambiente de desenvolvimento (DEV).
  - **Build:** Compilação do código e criação do artefato.
  - **GitLeaks:** Verificação de vazamento de segredos.
  - **Deploy to DEV:** Publicação do artefato no ambiente DEV para testes iniciais.

### 2. Branch Develop (`develop`)
- **Ação:** Recebe um Pull Request de uma branch `feature/*`.
- **Objetivo:** Após o merge, gerar um artefato entregável em DEV.
  - **Merge Validation:** Confirma que o PR está apto para ser mesclado (executa testes e validações).
  - **Build & Deploy:** Compilação do artefato após o merge e publicação em DEV.

### 3. Branch Release (`release/*`)
- **Ação:** É aberta a partir de uma branch `feature/*`.
- **Objetivo:** Gerar um artefato entregável em HML, podendo receber novos Pull Requests de branches `feature/*`.
  - **Build:** Compilação do código em um ambiente mais controlado.
  - **Deploy to HML:** Publicação do artefato no ambiente de homologação (HML).
  - **Additional PRs:** Permite a integração de novos PRs da branch `feature/*` para incorporar correções ou melhorias antes da entrega em produção.

### 4. Branch Master (`master`)
- **Ação:** Recebe Pull Request de uma branch `release/*`.
- **Objetivo:** Gerar o artefato final e entregá-lo em produção (PRD).
  - **Final QA Validation:** Execução de testes finais para garantir que o release está pronto para produção.
  - **Build for PRD:** Compilação final do artefato para produção.
  - **Deploy to PRD:** Publicação do artefato em produção.

### 5. Branch Hotfix (`hotfix/*`)
- **Ação:** É gerada a partir da master para correção imediata.
- **Objetivo:** Gerar um código que pode ser entregue diretamente em HML ou PRD.
  - **Hotfix Validation:** Testes rápidos para validar a correção.
  - **Deploy:** Entrega rápida do hotfix diretamente em HML ou PRD, dependendo da criticidade.

### 6. Branch de Pull Request
- **Ação:** Executa a stack de testes e SAST.
- **Objetivo:** Analisar a qualidade do PR sem gerar artefatos para publicação.
  - **Test Execution:** Execução de todos os testes automatizados para garantir que o PR não quebra nada.
  - **SAST (Static Application Security Testing):** Análise estática de segurança do código.
  - **PR Validation:** Relatórios de qualidade que atuam como gate para o merge.

# Opções do template

### Tipo de artefato [Artifact ou Container]
- `BuildType`
### Linguagem padrão [Java ou JS]
- `Language` 
  [''] - Padrão de execução é Java
### Cominho onde o artefato será publicado (Java, target/<artefato>.jar)
- `PathArtifact`
  [''] - Padrão é a raiz do repositorio
### Caminho onde o Dockerfile esta disponivel no repositorio
- `PathDockerfile`
  [''] - Padrão é a raiz do repositorio
### Habilita ou não a execução do Sonar
- `Sonar`
  [true] - Execução habilitada por padrão
### Habilita ou não a execução do VeraCode
- `Veracode`
  ["No"] - Execução habilitada por padrão
### Versão que o Node usará em tempo de BuildType
- `nodeVersion`
  ['21.5.0'] - Versão do node padrão
### Opção para habilitar ou desabilitar os testes do Maven
- `SkipTests`
  ['No'] - Padrão não desabilita os testes do build
### Habilita gate na execução do VeraCode
- `VeracodeLock`
  ['No'] - Não Bloqueia o gate do VeraCode por padrão
### Informa se o repositorio tem codigo fonte ou apenas o Dockerfile para build de imagem 
- `noCode`
  ['false'] - Por padrão o valor é desabilitado pois os repositorios contem codigo
### Habilita gate na execução do Sonar
- `SonarLock`
  ['0'] - Não bloqueia o gate de qualidade do Sonar
### Versão do Maven que será usada no Build
- `mavenVersion`
  ['3.5.4'] - Versão padrão do Maven
### Agente de build que será utilizado
- `poolName`
  ['Agents-Dev'] - Agent Pool padrão
### Possibilita a execução FULL em branches de longa duração [feature/, develop, release/, master]
- `fullExec`
  ['false'] - Valor padrão para execução completa (Sonar e Veracode)

  ---


## Parâmetros Configuráveis `(Ja mencionados acima)`

A pipeline é altamente customizável por meio de vários parâmetros, que permitem ajustes para diferentes tipos de projetos e fluxos de trabalho:

- **BuildType**: Define o tipo de build (ex.: Docker ou Maven).
- **Language**: Especifica a linguagem do projeto, influenciando as etapas subsequentes. As opções incluem **Java** e **JavaScript**.
- **PathArtifact**: Define o caminho para armazenar os artefatos gerados.
- **PathDockerfile**: Localização do Dockerfile utilizado no build.
- **Sonar**: Controle para habilitar ou desabilitar a análise de qualidade com **SonarQube**.
- **Veracode**: Se habilitado, realiza escaneamento de segurança com **Veracode**.
- **nodeVersion**: Define a versão do Node.js (relevante para projetos JavaScript).
- **SkipTests**: Configura para pular testes unitários se definido como "Yes".
- **SonarLock** e **VeracodeLock**: Controlam as travas de aprovação de qualidade e segurança.
- **mavenVersion**: Define a versão do Maven para builds em **Java**.
- **fullExec**: Parâmetro adicional que define a execução completa do build.
- **libArtifact**: Se configurado, define que artefatos de biblioteca serão publicados.

---

## Fluxo da Pipeline e Lógica de Decisão

### Estágios Principais

#### 1. Estágio de Configuração
- **Nome do Estágio**: Config
- **Condição**: Este estágio é ativado para branches como `feature`, `develop`, `release`, `master`, `hotfix`, e Pull Requests.
- **Função**: Configura os parâmetros de entrada e validações iniciais antes de prosseguir para as fases de build.

#### 2. Estágios de Build Baseados na Linguagem
- Dependendo do valor do parâmetro **Language**, a pipeline segue caminhos diferentes:

  - **JavaScript**: Executa o Gitleaks para verificação de segredos e constrói o projeto via o template `Workflow/CI/Lang/js.yaml`.
  - **Java**: Realiza o escaneamento de segurança via Veracode (se habilitado) e utiliza o Maven para construir o projeto com o template `Workflow/CI/Lang/java.yaml`.

---

## Artefatos Publicados

### Tipos de Artefatos
- **Artefatos de Build**: Incluem imagens Docker ou arquivos JAR/WAR, dependendo do tipo de projeto.
- **Resultados de Testes**: Se os testes forem executados (e não ignorados), os resultados são armazenados e publicados.
- **Relatórios de Segurança**: Incluem resultados de escaneamentos de segurança, como Veracode (se habilitado).

---

### Ferramentas CLI Utilizadas
- **Docker CLI**: Usado para construir e publicar imagens Docker.
- **Veracode CLI**: Utilizado para escaneamento de segurança (quando habilitado).
- **Maven CLI**: Usado para construir e gerenciar dependências em projetos Java.


## Fluxo de Decisões e Possibilidades da Pipeline

### Lógica de Decisão da Pipeline

A pipeline do template **newPipeWorkFlow** é construída com base em uma série de decisões lógicas que direcionam o fluxo conforme os parâmetros fornecidos e o tipo de branch. Cada estágio pode ou não ser acionado, dependendo dessas condições:

1. **Condição das Branches**: 
   - A pipeline verifica qual branch está sendo utilizada para determinar o fluxo. As principais branches que acionam os estágios são:
     - `feature/`
     - `develop/`
     - `release/`
     - `master/`
     - `hotfix/`
     - **Pull Requests**: Também acionam a execução da pipeline, porem com validações (veracode, Sonar, etc).

2. **Parâmetro de Linguagem**:
   - Dependendo do valor do parâmetro **Language**, a pipeline segue um fluxo específico:
     - **JavaScript**: Se a linguagem for definida como JavaScript, a pipeline segue pelo estágio de Gitleaks para verificar segredos no código. Posteriormente, o build é executado com o template `Workflow/CI/Lang/js.yaml`, onde o **Node.js** é configurado conforme a versão definida pelo parâmetro `nodeVersion`.
     - **Java**: Se a linguagem for Java, o fluxo segue para o escaneamento de segurança, se o **Veracode** estiver habilitado, e o build é executado com o **Maven**, utilizando o template `Workflow/CI/Lang/java.yaml`.

3. **Execução Completa ou Parcial**:
   - O parâmetro **fullExec** define se a pipeline será executada completamente ou se alguns estágios serão pulados (Isso serve para caso seja necessario a execução completa dos estagios nas branches principais, que originalmente só irão gerar o artefato de deploy).
   - Se **fullExec** for `false`, a pipeline pode ignorar alguns estágios, como a execução de testes ou o escaneamento de segurança quando executado nas branches principais.

4. **Análise de Qualidade e Segurança**:
   - Dependendo dos parâmetros **Sonar** e **Veracode**, a pipeline executa ou ignora a análise de qualidade e os escaneamentos de segurança.
     - **Sonar**: Se o parâmetro for `true`, o SonarQube será executado para verificar a qualidade do código.
     - **Veracode**: Se o parâmetro **Veracode** estiver habilitado, o escaneamento de segurança será realizado.
     - **Locks de Segurança**: As travas **SonarLock** e **VeracodeLock** definem se a pipeline deve parar caso uma dessas verificações falhe.

### Possibilidades de Configuração

1. **Linguagens Suportadas**:
   - **JavaScript**: A pipeline está otimizada para projetos JavaScript, com suporte a testes, verificação de segurança, e publicação de artefatos usando Node.js e Docker.
   - **Java**: Para projetos Java, a pipeline utiliza o **Maven** para build e gestão de dependências, com escaneamento de segurança opcional via Veracode.

2. **Suporte a Diferentes Ambientes**:
   - A pipeline pode ser adaptada para diferentes ambientes (como desenvolvimento, homologação, e produção) ajustando os parâmetros **BuildType**, **PathArtifact** e **PathDockerfile**, facilitando a implementação de múltiplos tipos de deploy.

3. **Personalização de Testes**:
   - Com o parâmetro **SkipTests**, a pipeline pode ser configurada para pular testes unitários quando não forem necessários, otimizando o tempo de execução.

### Exemplo de Fluxo de Decisão

- **Branch `feature/` + Linguagem JavaScript + Veracode habilitado**:
  - A pipeline executa os seguintes estágios:
    1. **Configuração dos Parâmetros**.
    2. **Verificação de Segredos com Gitleaks**.
    3. **Build do Projeto em Node.js**.
    4. **Escaneamento de Segurança com Veracode**.
    5. **Publicação dos Artefatos**.

- **Branch `release/` + Linguagem Java + Veracode desabilitado**:
  - A pipeline segue o fluxo:
    1. **Configuração dos Parâmetros**.
    2. **Build com Maven**.
    3. **Publicação dos Artefatos** (sem escaneamento de segurança).

---

## Comparação Final entre os Templates **Main** e **New**

### Template Main
- O **main** template inclui mais estágios opcionais, como um fluxo de pull do Git e verificações adicionais.
- Maior flexibilidade com relação a ambientes e condições adicionais para builds complexos.

### Template New
- O **<BRANCH>** template é mais otimizado e direto, removendo alguns estágios opcionais, mas mantendo a flexibilidade necessária para builds Java e JavaScript.
- Atualizações em versões de ferramentas e maior foco em modularidade e segurança, garantindo que as branches de longa e curta duração representem o que foi entregue no ambiente, e as branches de desenvolvimento sejam validadas com segurança, testes e analise de segurança.

Para migração para esse modelo é necessario ajustar por enquanto a referencia do resource de repositorio das pipelines, adicionando a flag: 
```
ref: feature/<BRANCH>
```

Para ajustar a pipeline de release para esse modelo é necessario apenas ajustar e ordenar os gatilhos de integração continua e de filtros de branches
---



