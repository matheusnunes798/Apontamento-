# Sistema de Controle de Produtividade Operacional — CD

## 1. Objetivo

Registrar com precisão o tempo que cada funcionário de um Centro de Distribuição
gasta em cada atividade operacional (separação, picking, conferência, embalagem,
recebimento, inventário), fornecendo dashboards em tempo real e relatórios
gerenciais. **Não é ponto eletrônico** — é controle de tempo por atividade.

## 2. Stack tecnológica e justificativa

| Camada | Tecnologia | Por quê |
|---|---|---|
| Backend | Node.js + TypeScript + NestJS | NestJS já implementa Clean Architecture / SOLID via módulos, injeção de dependência e camadas (controller → service → repository), reduzindo risco de inconsistência arquitetural ao longo do projeto |
| Banco | PostgreSQL | Relacional, forte em integridade referencial e concorrência — essencial para não permitir duas atividades abertas simultâneas |
| ORM | Prisma | Migrations versionadas, type-safety ponta a ponta com TS, seeds nativos |
| Frontend | React + TypeScript + Tailwind CSS | Ecossistema maduro, componentização, fácil responsividade (desktop/tablet/celular) e modo escuro via classes |
| Tempo real | WebSockets (Socket.IO) | Dashboard do gestor precisa refletir início/fim de atividades instantaneamente |
| Auth | JWT (access + refresh) | Padrão de mercado, permissões por role (admin/supervisor/funcionário) via guards do NestJS |
| Infra | Docker + docker-compose | Ambiente reproduzível, pronto para deploy |
| Testes | Jest (backend), Vitest (frontend) | Padrão de mercado nos respectivos ecossistemas |

Essa stack será mantida do início ao fim. Qualquer mudança será justificada
explicitamente antes de ser feita.

## 3. Estrutura de pastas (Clean Architecture)

```
cd-produtividade/
├── docker-compose.yml
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma
│   │   ├── migrations/
│   │   └── seed.ts
│   └── src/
│       ├── modules/
│       │   ├── auth/
│       │   ├── funcionarios/
│       │   ├── atividades/
│       │   ├── dashboard/
│       │   └── relatorios/
│       ├── domain/          # entidades e regras de negócio puras
│       ├── infra/           # implementações (repositórios Prisma, websocket gateway)
│       ├── common/          # guards, decorators, filters, interceptors
│       └── main.ts
└── frontend/
    └── src/
        ├── pages/
        ├── components/
        ├── hooks/
        ├── services/        # chamadas à API
        └── contexts/        # auth, tema
```

## 4. Plano de fases (ordem obrigatória do escopo original)

- **Fase 1 (esta entrega):** arquitetura, estrutura de pastas, modelagem do banco, migrations, diagrama de relacionamento
- **Fase 2:** módulo de autenticação (JWT + permissões) + módulo de funcionários (backend completo)
- **Fase 3:** módulo de atividades (iniciar/encerrar, regra de "uma ativa por vez", cálculo automático de duração)
- **Fase 4:** dashboard em tempo real (WebSocket) + endpoints agregados
- **Fase 5:** relatórios (diário/semanal/mensal/personalizado) + exportação PDF/Excel
- **Fase 6:** frontend (telas do funcionário e do gestor)
- **Fase 7:** testes automatizados
- **Fase 8:** Docker, docs de API, manuais de instalação/usuário/administrador

Cada fase será entregue com **código completo e funcional**, não pseudocódigo.
Vou avisar ao final de cada fase e seguir para a próxima — me diga se quiser
ajustar prioridade (ex.: começar pelo frontend, ou pular direto para o módulo
de atividades) a qualquer momento.
