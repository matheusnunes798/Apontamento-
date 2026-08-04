# Diagrama de Entidade-Relacionamento

Caminho: `ARQUITETURA.md` (raiz do projeto)

```mermaid
erDiagram
    SETOR ||--o{ FUNCIONARIO : possui
    SETOR ||--o{ EQUIPE : possui
    EQUIPE ||--o{ FUNCIONARIO : agrupa
    TURNO ||--o{ FUNCIONARIO : define
    FUNCIONARIO ||--o{ REGISTRO_ATIVIDADE : registra
    FUNCIONARIO ||--o{ LOG_AUDITORIA : gera
    FUNCIONARIO ||--o{ REFRESH_TOKEN : possui

    SETOR {
        uuid id PK
        string nome
    }

    EQUIPE {
        uuid id PK
        string nome
        uuid setorId FK
    }

    TURNO {
        uuid id PK
        string nome
        string horaInicio
        string horaFim
    }

    FUNCIONARIO {
        uuid id PK
        string nome
        string matricula UK
        string cargo
        string login UK
        string senhaHash
        enum permissao
        boolean ativo
        uuid setorId FK
        uuid equipeId FK
        uuid turnoId FK
    }

    REGISTRO_ATIVIDADE {
        uuid id PK
        uuid funcionarioId FK
        enum tipoAtividade
        datetime dataInicio
        datetime horaInicio
        datetime dataFim
        datetime horaFim
        int duracaoSegundos
        int quantidadePedidos
        int quantidadeItens
        string observacoes
        enum status
    }

    LOG_AUDITORIA {
        uuid id PK
        uuid funcionarioId FK
        enum acao
        string entidade
        uuid entidadeId
        json dadosAntes
        json dadosDepois
    }
```

## Decisões de modelagem

1. **`RegistroAtividade` é o coração do sistema.** Cada início/fim de
   atividade é uma linha. `duracaoSegundos` é calculado no momento do
   encerramento (nunca confiar em cálculo feito no front).
2. **Índice único parcial** (`WHERE status = 'ABERTO'`) impede duas
   atividades abertas para o mesmo funcionário — reforça no banco a regra
   que também será validada no service, evitando condição de corrida.
3. **`Equipe` e `Turno` são opcionais** no funcionário porque nem toda
   operação organiza por equipe/turno no dia 1, mas o campo já existe para
   os filtros do dashboard (`Data, Funcionário, Equipe, Setor, Turno`)
   pedidos no escopo.
4. **`LogAuditoria` é genérico** (`entidade` + `entidadeId` + JSON antes/depois)
   para não precisar de uma tabela de auditoria por entidade — cobre
   qualquer alteração futura sem migration nova.
5. **Relatórios não têm tabela própria.** São sempre calculados sob demanda
   a partir de `RegistroAtividade` (agregações), garantindo que nunca
   fiquem dessincronizados dos dados reais.
