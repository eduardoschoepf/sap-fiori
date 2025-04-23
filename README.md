# Visão Geral do Desenvolvimento RAP no Eclipse (ADT)
🔹 Interface CDS (ZI_...)
Define a estrutura de dados (campos, joins, associações).
Essa view normalmente é baseada em tabelas do sistema ou views reutilizáveis.

🔹 Consumption CDS (ZC_...)
View de consumo que expõe dados ao frontend, com anotações @UI, etc.
Pode incluir filtros, formatação, labels e lógica de apresentação.

🔹 Service Definition
Cria uma definição de serviço OData com base na view de consumo.
Exemplo: define service ZUI_MEUS_DADOS { expose ZC_MEUS_DADOS; }

🔹 Service Binding
Vincula o service definition a um canal de exposição (ex: OData V4).
Aqui o endpoint REST é gerado automaticamente.
→ Após ativar, você recebe a URL do serviço OData.

🔹 Behavior Definition (se for app transacional)
Define operações suportadas: create, update, delete, draft, validation, actions, etc.
Pode ser read-only ou implementar lógica customizada com classes ABAP.

🔹 Behavior Implementation (opcional)
Implementa a lógica ABAP (métodos como create, modify, delete, etc.).

🚀 Deploy e Teste no Fiori Launchpad
Com o serviço ativo e anotado corretamente, o app Fiori Elements pode ser consumido via Launchpad (FLP), App Manager, ou integrado em catálogos.
