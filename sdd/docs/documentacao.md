# Documentacao Tecnica - Mobile Green Wave

## 1. Visao Geral

O Mobile Green Wave e um aplicativo mobile multiplataforma, para Android e iOS, voltado a orientar motoristas em tempo real sobre a velocidade mais adequada para aumentar a chance de atravessar sinais verdes ao longo do trajeto.

O produto combina dados de GPS, direcao da via, distancia ate o semaforo e ciclos semaforicos para calcular recomendacoes de velocidade minima, media e maxima, sempre respeitando seguranca viaria e limite legal da via.

Quando os dados de ciclos semaforicos nao estiverem disponiveis ou estiverem incompletos, o app deve permitir cadastro e complementacao colaborativa dessas informacoes por cruzamento, mantendo autoria, timestamp, auditoria e nivel de confianca.

Esta documentacao substitui a documentacao legado de outro projeto e passa a descrever exclusivamente o escopo do aplicativo definido em [sdd/01-spec.md](sdd/01-spec.md) e [sdd/02-plan.md](sdd/02-plan.md).

## 2. Objetivo do Produto

O aplicativo deve:

- orientar o motorista com base em mapa GPS e dados semaforicos;
- recomendar velocidade minima, media e maxima para o proximo semaforo;
- evoluir para recomendacao de sequencia de semaforos no trajeto;
- permitir inclusao e atualizacao colaborativa de ciclos semaforicos;
- operar com resiliencia em cenarios de perda de rede, timeout, baixa precisao de GPS e inconsistencias de dados.

## 3. Personas Atendidas

- Motorista urbano diario, com foco em previsibilidade e economia.
- Motorista profissional, com foco em eficiencia operacional.
- Motorista eventual em area urbana, com foco em orientacao simples e segura.

## 4. Funcionalidades Principais

### 4.1 Navegacao orientada por semaforos

Com base na localizacao do usuario e nos dados do cruzamento mais relevante a frente, o app deve calcular uma faixa de velocidade recomendada para favorecer a travessia em sinal verde.

Entradas minimas esperadas:

- posicao GPS valida;
- direcao da via;
- distancia ate o semaforo;
- fase ou ciclo do sinal;
- limite de velocidade da via.

Saidas previstas:

- velocidade minima sugerida;
- velocidade media sugerida;
- velocidade maxima sugerida;
- indicacao de confiabilidade da recomendacao;
- suspensao explicita da recomendacao quando os dados nao forem confiaveis.

### 4.2 Inclusao e complementacao colaborativa de ciclos

Na ausencia de dados consolidados, o motorista colaborador deve conseguir cadastrar e complementar os momentos de abertura e fechamento dos sinais por cruzamento e direcao.

Cada submissao deve registrar:

- cruzamento;
- direcao;
- horarios informados;
- usuario responsavel;
- timestamp da operacao;
- nivel de confianca;
- status de validacao.

## 5. Regras de Negocio Invariantes

As seguintes regras sao obrigatorias e tem precedencia sobre qualquer tentativa de otimizacao:

- o sistema nunca deve recomendar conduta que implique avancar sinal vermelho;
- toda recomendacao deve respeitar o limite legal da via;
- recomendacoes so podem ser exibidas com dados minimos validos;
- na ausencia de dados confiaveis, a recomendacao deve ser suspensa;
- cadastros manuais precisam ser auditaveis e reversiveis;
- o app deve priorizar seguranca viaria sobre ganho de tempo ou economia de combustivel.

## 6. Escopo Tecnico da Solucao

### 6.1 Stack principal

- React Native com TypeScript;
- preferencia por Expo com EAS Build para empacotamento Android e iOS;
- React Navigation para navegacao;
- react-native-maps para exibicao de mapa e sinais;
- Axios para integracoes HTTP com timeout, retry e interceptors;
- AsyncStorage para cache local e rascunhos;
- React Hook Form com Zod para formularios e validacao;
- Jest e React Native Testing Library para testes;
- ESLint e Prettier para padronizacao.

### 6.2 Principios de arquitetura

- componentes devem ficar focados em apresentacao e orquestracao leve;
- regras de negocio e casos de uso devem ficar em services;
- logica de dominio nao deve ser embutida em telas ou componentes visuais;
- constantes compartilhadas devem ficar em `src/app/constants/constants.ts`;
- utilitarios transversais devem ficar em `src/app/utils`.

## 7. Arquitetura Logica

### 7.1 Camadas previstas

- Models: representam entidades e contratos consumidos pelo app.
- Pages: concentram as telas e fluxos da aplicacao.
- Services: implementam regras de negocio, integracoes e orquestracao tecnica.
- Shared: concentra componentes, hooks, estado, validadores e tema.
- Utils: contem funcoes utilitarias puras e reutilizaveis.

### 7.2 Services principais

- `RecommendationService`: calcula velocidade minima, media e maxima para o proximo sinal e, futuramente, para sequencias de sinais.
- `SafetyGuardService`: valida invariantes de seguranca e pode bloquear recomendacoes inconsistentes.
- `SignalCycleService`: consulta, cria e atualiza ciclos semaforicos.
- `MapService`: obtem regioes e sinais mapeados.
- `GpsService`: fornece posicao, direcao e precisao do usuario.
- `DraftService`: salva e recupera rascunhos localmente.
- `AuditService`: registra autoria, timestamp, versao e historico logico das alteracoes.

## 8. Estrutura de Diretorios Prevista

```text
src/
    app/
        constants/
            constants.ts
        models/
            profile.model.ts
            map.model.ts
            traffic-light.model.ts
            signal-cycle.model.ts
            recommendation.model.ts
            api-error.model.ts
        pages/
            home/
            guided-navigation/
            signal-create/
            signal-update/
            settings/
        services/
            api/
                http-client.ts
                profile.api.ts
                map.api.ts
                signal-cycle.api.ts
            gps/
                gps.service.ts
            map/
                map.service.ts
            recommendation/
                recommendation.service.ts
            safety/
                safety-guard.service.ts
            draft/
                draft.service.ts
            audit/
                audit.service.ts
        shared/
            components/
            hooks/
            state/
            validators/
            theme/
        utils/
            date-time.ts
            geo.ts
            math.ts
tests/
    unit/
    integration/
```

## 9. Fluxos Funcionais Esperados

### 9.1 Fluxo de navegacao orientada

1. O app obtem a posicao atual do usuario.
2. O app identifica a direcao da via e o cruzamento mais relevante a frente.
3. O app consulta dados de sinais da regiao ou usa cache disponivel.
4. O `RecommendationService` calcula a faixa de velocidade recomendada.
5. O `SafetyGuardService` valida se a recomendacao atende as regras de seguranca.
6. A interface exibe a recomendacao ou informa suspensao por baixa confiabilidade.

### 9.2 Fluxo de cadastro colaborativo

1. O usuario seleciona um cruzamento e a direcao.
2. O usuario informa horarios de abertura e fechamento do ciclo.
3. O formulario valida os campos localmente.
4. O rascunho e salvo automaticamente durante a edicao.
5. O app envia os dados para a API quando houver conectividade.
6. Em sucesso, o rascunho e removido e o usuario recebe confirmacao.
7. Em erro, o formulario permanece editavel e o rascunho e preservado.

## 10. Contratos Minimos de API

Todas as APIs devem padronizar o payload de erro no formato abaixo:

```json
{
    "code": "string",
    "message": "string",
    "details": {},
    "traceId": "string"
}
```

### 10.1 Listar perfis do cliente

- Metodo: `GET`
- Rota: `/v1/clients/profiles?login={login}`

Resposta esperada:

```json
{
    "profiles": [
        {
            "id": "string",
            "clientId": "string",
            "name": "string",
            "role": "string"
        }
    ]
}
```

### 10.2 Obter roteiros padrao de sinais mapeados

- Metodo: `GET`
- Rota: `/v1/clients/{clientId}/signal-routes`

Resposta esperada:

```json
{
    "routes": [
        {
            "id": "string",
            "name": "string",
            "regionId": "string",
            "intersections": ["string"]
        }
    ]
}
```

### 10.3 Obter sinais de uma regiao no mapa

- Metodo: `GET`
- Rota: `/v1/maps/regions/{regionId}/signals?bbox={minLng,minLat,maxLng,maxLat}`

Resposta esperada:

```json
{
    "intersections": [
        {
            "id": "string",
            "lat": 0,
            "lng": 0,
            "roadName": "string",
            "direction": "string",
            "speedLimitKmh": 60,
            "confidenceLevel": "string",
            "cycle": {
                "greenWindows": [
                    {
                        "start": "string",
                        "end": "string"
                    }
                ],
                "redWindows": [
                    {
                        "start": "string",
                        "end": "string"
                    }
                ],
                "source": "string",
                "updatedAt": "string"
            }
        }
    ]
}
```

### 10.4 Criar ciclo semaforico

- Metodo: `POST`
- Rota: `/v1/signals/cycles`

Body minimo:

```json
{
    "intersectionId": "string",
    "direction": "string",
    "greenStart": "string",
    "greenEnd": "string",
    "redStart": "string",
    "redEnd": "string",
    "timezone": "string",
    "submittedBy": "string"
}
```

Resposta esperada:

```json
{
    "id": "string",
    "status": "PENDING_VALIDATION",
    "confidenceLevel": "string",
    "createdAt": "string"
}
```

### 10.5 Atualizar ciclo semaforico

- Metodo: `PATCH`
- Rota: `/v1/signals/cycles/{cycleId}`

Body minimo:

```json
{
    "greenStart": "string",
    "greenEnd": "string",
    "redStart": "string",
    "redEnd": "string",
    "reason": "string",
    "updatedBy": "string"
}
```

Resposta esperada:

```json
{
    "id": "string",
    "status": "string",
    "confidenceLevel": "string",
    "version": 1,
    "updatedAt": "string"
}
```

## 11. Resiliencia e Tratamento de Erros

### 11.1 Politica minima de resiliencia

- timeout de API de 8 segundos;
- ate 2 retentativas exponenciais;
- uso do ultimo cache valido quando a integracao falhar repetidamente;
- operacao em modo degradado quando nao houver conectividade;
- bloqueio de envio online enquanto nao houver reconexao.

### 11.2 Cenarios criticos previstos

- perda de GPS ou precisao insuficiente: pausar recomendacoes;
- timeout de API: tentar novamente e, se necessario, usar cache com aviso;
- semaforo inconsistente ou conflitante: marcar cruzamento como nao confiavel;
- payload invalido: bloquear envio no cliente e destacar campos;
- erro 4xx retornado pelo backend: exibir mensagens acionaveis por campo.

### 11.3 Regras de interface

O botao de envio deve permanecer bloqueado quando:

- o formulario estiver invalido;
- o envio estiver em andamento;
- nao houver conectividade para a operacao online;
- os dados minimos ainda nao tiverem sido preenchidos.

Comportamentos obrigatorios:

- mensagens de erro claras e acionaveis;
- autosave de rascunho durante edicao;
- preservacao do rascunho em caso de erro;
- limpeza do rascunho em caso de sucesso.

## 12. Consideracoes de Seguranca e Confiabilidade

- o app nao pode incentivar comportamento inseguro no transito;
- recomendacoes devem ser suspensas automaticamente em caso de baixa confiabilidade;
- os dados colaborativos devem ser versionados e auditaveis;
- informacoes de autoria devem ser preservadas em criacoes e alteracoes;
- a origem dos ciclos semaforicos deve ser rastreavel.

## 13. Estrategia de Testes

### 13.1 Tipos de teste

- testes unitarios para servicos de recomendacao, seguranca e confiabilidade;
- testes de integracao para contratos de API e tratamento de erros;
- testes de UI para estados de loading, erro, sucesso, vazio e modo degradado.

### 13.2 Meta inicial de cobertura

- cobertura minima de 80% em `src/app/services`.

### 13.3 Cenarios obrigatorios

- perda de GPS;
- timeout da API;
- semaforo inconsistente;
- operacao offline;
- cadastro colaborativo com validacao de campos.

## 14. Plano de Entrega Incremental

### Fase 1

Setup da aplicacao, navegacao base, mapa e integracao de GPS.

### Fase 2

Leitura de sinais mapeados e recomendacao para o proximo semaforo.

### Fase 3

Recomendacao para sequencia de semaforos e exibicao de confiabilidade.

### Fase 4

Inclusao e complementacao colaborativa de ciclos, com auditoria.

### Fase 5

Resiliencia completa, cache, retry, offline, testes e preparacao de release Android e iOS.

## 15. Ambiente de Desenvolvimento Recomendado

Como diretriz para implementacao do app, o ambiente local deve contemplar:

- Node.js em versao LTS atual;
- npm ou gerenciador de pacotes equivalente;
- Expo CLI por uso via `npx expo`;
- Android Studio para emulacao Android;
- Xcode para execucao iOS em macOS;
- EAS CLI para empacotamento e distribuicao quando necessario.

Fluxo esperado de inicializacao do projeto apos o bootstrap da base React Native/Expo:

```bash
npm install
npx expo start
```

Para execucao nativa, quando aplicavel:

```bash
npx expo run:android
npx expo run:ios
```

Observacao: este repositorio ainda esta orientado por artefatos SDD. Os comandos acima representam o fluxo esperado para a base do app quando a estrutura de codigo estiver criada.

## 16. Conclusao

O Mobile Green Wave e uma solucao mobile orientada por seguranca, previsibilidade e eficiencia urbana. A arquitetura proposta separa interface, dominio e integracoes, reduz acoplamento e prepara o produto para evolucao incremental.

Esta documentacao deve servir como referencia operacional para a implementacao do app e deve permanecer alinhada aos artefatos de especificacao e plano tecnico em [sdd/01-spec.md](sdd/01-spec.md) e [sdd/02-plan.md](sdd/02-plan.md).