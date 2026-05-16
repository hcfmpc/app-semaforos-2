1. Objetivo técnico
Implementar um aplicativo mobile multiplataforma (Android/iOS) com React Native (TypeScript), com duas funcionalidades centrais:

- Inclusão de Momentos dos Sinais:
  Permitir cadastro e complementação colaborativa de ciclos semafóricos por cruzamento (abertura/fechamento), registrando usuário, timestamp e nível de confiança.
- Navegação Orientada:
  Com base em GPS, direção da via, distância ao semáforo e fase/ciclo, recomendar velocidade mínima, média e máxima para aumentar a chance de atravessar no verde, sempre respeitando limite legal e segurança viária.

2. Stack Tecnológico e padrões
- React Native + TypeScript (preferencialmente Expo + EAS Build para build nativo Android/iOS).
- React Navigation para navegação.
- react-native-maps para mapa e camadas de sinais.
- Axios para HTTP (timeout, retry e interceptors).
- AsyncStorage para cache e rascunhos locais.
- React Hook Form + Zod para formulários e validação.
- Jest + React Native Testing Library para testes.
- ESLint + Prettier para padronização.

Padrões de arquitetura:
- Componentes focam em apresentação e orquestração leve.
- Regras de negócio e casos de uso ficam em services.
- Constantes em `src/app/constants/constants.ts`.
- Utilitários em `src/app/utils`.
- Organização:
  - `src/app/models` (modelos)
  - `src/app/pages` (telas/fluxos)
  - `src/app/services` (casos de uso/regras)
  - `src/app/shared` (reuso)
- Evitar lógica de negócio em componentes.

3. Integrações de API (contratos mínimos)
Definir contratos REST (placeholders) com payload de erro padrão:
`{ code: string, message: string, details?: any, traceId?: string }`

3.1 Listar perfis do cliente (por login)
- Método: `GET`
- Rota: `/v1/clients/profiles?login={login}`
- Resposta 200:
  - `profiles: Array<{ id, clientId, name, role }>`.

3.2 Obter roteiros padrão de sinais já mapeados pelo cliente
- Método: `GET`
- Rota: `/v1/clients/{clientId}/signal-routes`
- Resposta 200:
  - `routes: Array<{ id, name, regionId, intersections: string[] }>`.

3.3 Obter mapa da região contendo sinais já mapeados
- Método: `GET`
- Rota: `/v1/maps/regions/{regionId}/signals?bbox={minLng,minLat,maxLng,maxLat}`
- Resposta 200:
  - `intersections: Array<{ id, lat, lng, roadName, direction, speedLimitKmh, confidenceLevel, cycle }>`
  - `cycle: { greenWindows: Array<{ start, end }>, redWindows: Array<{ start, end }>, source, updatedAt }`.

3.4 Enviar inclusão (criação) de tempos de sinais
- Método: `POST`
- Rota: `/v1/signals/cycles`
- Body mínimo:
  - `{ intersectionId, direction, greenStart, greenEnd, redStart, redEnd, timezone, submittedBy }`
- Resposta 201:
  - `{ id, status: "PENDING_VALIDATION" | "VALIDATED", confidenceLevel, createdAt }`.

3.5 Enviar complementação (regularização/atualização) de tempos de sinais
- Método: `PATCH`
- Rota: `/v1/signals/cycles/{cycleId}`
- Body mínimo:
  - `{ greenStart?, greenEnd?, redStart?, redEnd?, reason, updatedBy }`
- Resposta 200:
  - `{ id, status, confidenceLevel, version, updatedAt }`.

4. Erros e resiliência (mínimo)
Comportamento padrão:
- Timeout de API: 8s com até 2 retentativas exponenciais; persistindo falha, usar último cache com aviso de desatualização.
- Falha de rede/offline: operar em modo degradado com cache; bloquear envio online até reconexão.
- Validação do backend (4xx): exibir mensagens por campo e manter formulário editável.
- Payload inválido no cliente: bloquear envio e destacar campos inválidos.
- GPS indisponível/baixa precisão: suspender recomendações até recuperação mínima.
- Dados semafóricos inconsistentes/conflitantes: marcar cruzamento como não confiável e não recomendar automaticamente.

Regras de UI:
- Botão **Enviar** bloqueado quando:
  - formulário inválido;
  - envio em progresso;
  - sem conectividade para operação online;
  - dados mínimos ausentes.
- Mensagens de erro devem ser claras e acionáveis.
- Rascunho:
  - autosave local durante edição;
  - em erro, manter rascunho;
  - em sucesso, limpar rascunho e exibir confirmação.

5. Casos de uso técnicos (services)
- `RecommendationService`: calcula velocidade min/méd/máx para próximo sinal e sequência.
- `SafetyGuardService`: garante invariantes (nunca avançar vermelho, respeitar limite legal, suspender com baixa confiabilidade).
- `SignalCycleService`: consulta, cria e atualiza ciclos semafóricos.
- `MapService`: obtém mapa/região e sinais.
- `GpsService`: posição, direção e precisão.
- `DraftService`: persistência e recuperação de rascunhos.
- `AuditService`: autoria, timestamp, versão e reversão lógica de alterações.

6. Estratégia de testes
- Unitários: regras em `services` (recomendação, segurança e confiabilidade).
- Integração: contratos de API e tratamento de erros.
- UI/telas: estados de loading, erro, sucesso, vazio e modo degradado.
- Cobertura mínima inicial: 80% para `src/app/services`.
- Cenários críticos:
  - perda de GPS;
  - timeout da API;
  - semáforo inconsistente;
  - operação offline;
  - cadastro colaborativo com validação de campos.

7. Plano de entrega (MVP incremental)
- Fase 1: setup do app, navegação, mapa e GPS.
- Fase 2: leitura de sinais mapeados + recomendação para próximo semáforo.
- Fase 3: recomendação para sequência de semáforos + indicador de confiabilidade.
- Fase 4: inclusão/complementação colaborativa + auditoria.
- Fase 5: resiliência (cache/retry/offline), testes e preparação de release Android/iOS.

8. Estrutura de Diretórios
- `src/`
  - `app/`
    - `constants/`
      - `constants.ts`
    - `models/`
      - `profile.model.ts`
      - `map.model.ts`
      - `traffic-light.model.ts`
      - `signal-cycle.model.ts`
      - `recommendation.model.ts`
      - `api-error.model.ts`
    - `pages/`
      - `home/`
      - `guided-navigation/`
      - `signal-create/`
      - `signal-update/`
      - `settings/`
    - `services/`
      - `api/`
        - `http-client.ts`
        - `profile.api.ts`
        - `map.api.ts`
        - `signal-cycle.api.ts`
      - `gps/`
        - `gps.service.ts`
      - `map/`
        - `map.service.ts`
      - `recommendation/`
        - `recommendation.service.ts`
      - `safety/`
        - `safety-guard.service.ts`
      - `draft/`
        - `draft.service.ts`
      - `audit/`
        - `audit.service.ts`
    - `shared/`
      - `components/`
      - `hooks/`
      - `state/`
      - `validators/`
      - `theme/`
    - `utils/`
      - `date-time.ts`
      - `geo.ts`
      - `math.ts`
- `tests/`
  - `unit/`
  - `integration/`