# 🚀 Guia Prático de Desenvolvimento RAP no Eclipse (ADT) com Fiori Elements

Este documento é um guia educacional sobre como criar aplicações **SAP Fiori Elements** usando o **RAP (Restful ABAP Programming Model)** no **ABAP Development Tools (ADT)**.

---

## 📖 1. Introdução

O **RAP** é o modelo de programação ABAP orientado a serviços, projetado para criar APIs e aplicações SAP Fiori de forma declarativa, escalável e reutilizável.  
Ele permite:
- Criar **CDS Views** para modelagem de dados
- Definir **comportamento** com Behavior Definitions
- Expor dados via **OData Services**
- Configurar **UI** diretamente no backend com anotações

No desenvolvimento RAP com Fiori Elements, seguimos normalmente esta ordem:

1. **Modelar os dados** (Interface e Consumption Views)
2. **Definir apresentação** (Metadata Extensions)
3. **Expor serviço** (Service Definition + Service Binding)
4. **Definir comportamento** (Behavior Definition/Implementation)
5. **Testar e publicar** (Fiori Launchpad/App Manager)

---

## 📂 2. Modelagem de Dados (CDS Views)

As CDS Views (Core Data Services) representam a camada de dados e podem conter **anotações** que afetam como os dados serão exibidos e consumidos.

### 2.1 Interface View (ZI\_...)
Camada base que encapsula tabelas, define associações e estrutura principal dos dados.  
Não é exposta diretamente ao frontend.

**Exemplo – Cabeçalho:**
```abap
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Ordem de Vendas - Cabeçalho - Eduardo'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{ serviceQuality: #X, sizeCategory: #S, dataClass: #MIXED }
@Search.searchable: true

define root view entity ZI_HEADEROV_ESCHOEPF 
    as select from I_SalesOrder
    composition [1..*] of ZI_ITEMOV_ESCHOEPF as _SalesOrderItem
{
    @Search.defaultSearchElement: true
    key SalesOrder           as SalesOrder,
        SalesOrderType       as SalesOrderType,
        CreatedByUser        as CreatedByUser,
        LastChangedByUser    as LastChangedByUser,
        CreationDate         as CreationDate,
        SalesOrganization    as SalesOrganization,
        DistributionChannel  as DistributionChannel,
        OrganizationDivision as OrganizationDivision,
        SalesGroup           as SalesGroup,
        SalesOffice          as SalesOffice,
        @Semantics.amount.currencyCode: 'TransactionCurrency'
        TotalNetAmount       as TotalNetAmount,
        TransactionCurrency  as TransactionCurrency,
        
        case
          when  TotalNetAmount  > 10000 then '3' -- Verde
          when  TotalNetAmount  > 0 then '2'     -- Amarelo
          else '1'                               -- Vermelho
        end as Criticality,
        
        _SalesOrderItem
}

```

**Exemplo – Item:**
```abap
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Ordem de vendas - Item - Eduardo'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZI_ITEMOV_ESCHOEPF 
    as select from I_SalesOrderItem 
    association to parent ZI_HEADEROV_ESCHOEPF as _SalesOrder
    on $projection.SalesOrder = _SalesOrder.SalesOrder
{
    key SalesOrder             as SalesOrder,
    key SalesOrderItem         as SalesOrderItem,
        SalesOrderItemCategory as SalesOrderItemCategory,
        SalesOrderItemType     as SalesOrderItemType,
        IsReturnsItem          as IsReturnsItem,
        CreatedByUser          as CreatedByUser,
        CreationDate           as CreationDate,
        CreationTime           as CreationTime,
        LastChangeDate         as LastChangeDate,
        Division               as Division,
        Material               as Material,
        Batch                  as Batch,
        Plant                  as Plant,
        StorageLocation        as StorageLocation,
        @Semantics.quantity.unitOfMeasure: 'OrderQuantityUnit'
        OrderQuantity          as OrderQuantity, 
        OrderQuantityUnit      as OrderQuantityUnit,
        @Semantics.quantity.unitOfMeasure: 'ItemWeightUnit'
        ItemGrossWeight        as ItemGrossWeight,
        @Semantics.quantity.unitOfMeasure: 'ItemWeightUnit'
        ItemNetWeight          as ItemNetWeight,
        ItemWeightUnit         as ItemWeightUnit,
        @Semantics.quantity.unitOfMeasure: 'ItemVolumeUnit'
        ItemVolume             as ItemVolume,
        ItemVolumeUnit         as ItemVolumeUnit,
        @Semantics.amount.currencyCode: 'TransactionCurrency'
        NetAmount              as NetAmount,
        TransactionCurrency    as TransactionCurrency,
      
        _SalesOrder    
       
}
```

### 2.2 Consumption View (ZC_...)

Camada de projeção que expõe dados ao frontend (Fiori Elements), contendo anotações para UI, formatação e filtragem.  

**Exemplo – Cabeçalho:**
```abap
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Ordem de Venda - Cabeçalho - ESCHOEPF'
@Metadata.ignorePropagatedAnnotations: false
@Metadata.allowExtensions: true
define root view entity ZC_HEADEROV_ESCHOEPF
  provider contract transactional_query
    as projection on ZI_HEADEROV_ESCHOEPF
{
    @ObjectModel.text.element: [ 'SalesOrderType' ]
    @UI.textArrangement: #TEXT_LAST
    key SalesOrder           as SalesOrder,
        SalesOrderType       as SalesOrderType,
        CreatedByUser        as CreatedByUser,
        LastChangedByUser    as LastChangedByUser,
        CreationDate         as CreationDate,
        SalesOrganization    as SalesOrganization,
        DistributionChannel  as DistributionChannel,
        OrganizationDivision as OrganizationDivision,
        SalesGroup           as SalesGroup,
        SalesOffice          as SalesOffice,   
        TotalNetAmount       as TotalNetAmount,
        TransactionCurrency  as TransactionCurrency,
        Criticality          as Criticality, 
        
        _SalesOrderItem : redirected to composition child ZC_ITEMOV_ESCHOEPF
}
```
**Exemplo – Item:**
```abap
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Ordem de Vendas - Item -ESCHOEPF'
@Metadata.ignorePropagatedAnnotations: false
@Metadata.allowExtensions: true

define view entity ZC_ITEMOV_ESCHOEPF 
    as projection on ZI_ITEMOV_ESCHOEPF
{
    key SalesOrder             as SalesOrder,
    key SalesOrderItem         as SalesOrderItem,
        SalesOrderItemCategory as SalesOrderItemCategory,
        SalesOrderItemType     as SalesOrderItemType,
        IsReturnsItem          as IsReturnsItem,
        CreatedByUser          as CreatedByUser,
        CreationDate           as CreationDate,
        CreationTime           as CreationTime,
        LastChangeDate         as LastChangeDate,
        Division               as Division,
        Material               as Material,
        Batch                  as Batch,
        Plant                  as Plant,
        StorageLocation        as StorageLocation,
        @Semantics.quantity.unitOfMeasure: 'OrderQuantityUnit'
        OrderQuantity          as OrderQuantity, 
        OrderQuantityUnit      as OrderQuantityUnit,
        @Semantics.quantity.unitOfMeasure: 'ItemWeightUnit'
        ItemGrossWeight        as ItemGrossWeight,
        @Semantics.quantity.unitOfMeasure: 'ItemWeightUnit'
        ItemNetWeight          as ItemNetWeight,
        ItemWeightUnit         as ItemWeightUnit,
        @Semantics.quantity.unitOfMeasure: 'ItemVolumeUnit'
        ItemVolume             as ItemVolume,
        ItemVolumeUnit         as ItemVolumeUnit,
        @Semantics.amount.currencyCode: 'TransactionCurrency'
        NetAmount              as NetAmount,
        TransactionCurrency    as TransactionCurrency,
      
        _SalesOrder   : redirected to parent ZC_HEADEROV_ESCHOEPF
}
```  

### 🎨 3. Configuração de UI (Metadata Extensions)

Usada para modularizar e centralizar a configuração da interface no Fiori Elements, sem poluir a CDS View principal.

📌 Recomendada para manter o código limpo e reutilizável.

**Exemplo – Cabeçalho:**
```abap
@Metadata.layer: #CORE
@UI.headerInfo: { typeName: 'Pedidos de compra',
                  typeNamePlural: 'Pedidos de compra' }
                 
@UI.presentationVariant: [{ sortOrder: [{by: 'TotalNetAmount', direction: #DESC }] }]
                  
annotate entity ZC_HEADEROV_ESCHOEPF
    with 
{
    
    @UI.facet: [{
        id: 'header',
        label: 'Cabeçalho',
        position: 10,
        type: #FIELDGROUP_REFERENCE, //ou #IDENTIFICATION_REFERENCE
        targetQualifier: 'HEADER'
    }, {
        id: 'items',                        // Adiciona a faceta para os itens
        label: 'Itens da Ordem de Vendas',  // Nome da faceta
        position: 20,                       // A posição da faceta na UI
        type: #LINEITEM_REFERENCE,
        targetElement: '_SalesOrderItem'    // Qualificador de campo para os itens
    }]
     
    // Campos do cabeçalho da ordem de vendas
    //@Consumption.filter.mandatory: true         
    @UI.lineItem: [{ position: 10, label: 'Número da Ordem de Vendas' }]
    @UI.selectionField: [{ position: 10 }]
    @EndUserText.label: 'Pedido de compras'
    @Consumption.valueHelpDefinition: [{ entity.name: 'I_MM_SalesOrderItemVH', entity.element: 'SalesOrder' }]
    @UI.fieldGroup: [{ position: 10, qualifier: 'HEADER' }]  // Posição 10, agrupando no cabeçalho
    SalesOrder;

    @UI.selectionField: [{ position: 20 }]
    @UI.lineItem: [{ position: 20 }]
    @Consumption.valueHelpDefinition: [{ entity.name: 'I_SalesOrderType', entity.element: 'SalesOrderType' }]
    @UI.fieldGroup: [{ position: 20, qualifier: 'HEADER' }]  // Posição 20, agrupando no cabeçalho
    SalesOrderType;

    @UI.lineItem: [{ position: 30 }]
    @UI.fieldGroup: [{ position: 30, qualifier: 'HEADER' }]  // Posição 30, agrupando no cabeçalho
    CreatedByUser;

    @UI.lineItem: [{ position: 40 }]
    @UI.fieldGroup: [{ position: 40, qualifier: 'HEADER' }]  // Posição 40, agrupando no cabeçalho
    LastChangedByUser;

    @UI.selectionField: [{ position: 30 }]
    @UI.lineItem: [{ position: 50 }]
    @Consumption.filter: { selectionType: #RANGE }
    @UI.fieldGroup: [{ position: 50, qualifier: 'HEADER' }]  // Posição 50, agrupando no cabeçalho
    CreationDate;

    @UI.lineItem: [{ position: 60 }]
    @UI.fieldGroup: [{ position: 60, qualifier: 'HEADER' }]  // Posição 60, agrupando no cabeçalho
    SalesOrganization;

    @UI.lineItem: [{ position: 70 }]
    @UI.fieldGroup: [{ position: 70, qualifier: 'HEADER' }]  // Posição 70, agrupando no cabeçalho
    DistributionChannel;

    @UI.lineItem: [{ position: 80 }]
    @UI.fieldGroup: [{ position: 80, qualifier: 'HEADER' }]  // Posição 80, agrupando no cabeçalho
    OrganizationDivision;

    @UI.lineItem: [{ position: 90 }]
    @UI.fieldGroup: [{ position: 90, qualifier: 'HEADER' }]  // Posição 90, agrupando no cabeçalho
    SalesGroup;

    @UI.lineItem: [{ position: 100 }]
    @UI.fieldGroup: [{ position: 100, qualifier: 'HEADER' }]  // Posição 100, agrupando no cabeçalho
    SalesOffice;
    
    @UI.lineItem: [{ position: 110, criticality: 'Criticality' }]
    @UI.fieldGroup: [{ position: 110, qualifier: 'HEADER', criticality: 'Criticality' }]  // Posição 110, agrupando no cabeçalho
    TotalNetAmount;
    
    @UI.lineItem: [{ position: 120 }]
    @UI.hidden: true
    @UI.fieldGroup: [{ position: 120, qualifier: 'HEADER' }]  // Posição 120, agrupando no cabeçalho
    TransactionCurrency;
    
    // Faceta para os itens da ordem de vendas
    @UI.fieldGroup: [{ position: 140, qualifier: 'ITEMS' }]  // Posição 140 para os itens da ordem de vendas
    _SalesOrderItem; // Esta é a associação que traz os itens da ordem de vendas
    
}
```

**Exemplo – Item:**
```abap
@Metadata.layer: #CORE
annotate entity ZC_ITEMOV_ESCHOEPF with
{
      @UI.lineItem: [{ position: 10 }]
      SalesOrder;
      @UI.lineItem: [{ position: 20 }]
      SalesOrderItem;
      @UI.lineItem: [{ position: 30 }]
      SalesOrderItemCategory;
      @UI.lineItem: [{ position: 40 }]
      SalesOrderItemType;
      @UI.lineItem: [{ position: 50 }]
      IsReturnsItem;
      @UI.lineItem: [{ position: 60 }]
      CreatedByUser;
      @UI.lineItem: [{ position: 70 }]
      CreationDate;
      @UI.lineItem: [{ position: 80 }]
      CreationTime;
      @UI.lineItem: [{ position: 90 }]
      LastChangeDate;
      @UI.lineItem: [{ position: 100 }]
      Division;
      @UI.lineItem: [{ position: 110 }]
      Material;
      @UI.lineItem: [{ position: 120 }]
      Batch;
      @UI.lineItem: [{ position: 130 }]
      Plant;
      @UI.lineItem: [{ position: 140 }]
      StorageLocation;
      @UI.lineItem: [{ position: 150 }]
      OrderQuantity;
      @UI.lineItem: [{ position: 160 }]
      OrderQuantityUnit;
      @UI.lineItem: [{ position: 170 }]
      ItemGrossWeight;
      @UI.lineItem: [{ position: 180 }]
      ItemNetWeight;
      @UI.lineItem: [{ position: 190 }]
      ItemWeightUnit;
      @UI.lineItem: [{ position: 200 }]
      ItemVolume;
      @UI.lineItem: [{ position: 210 }]
      ItemVolumeUnit;
      @UI.lineItem: [{ position: 220 }]
      NetAmount;
      @UI.lineItem: [{ position: 230 }]
      TransactionCurrency;
}
```

### 🌐 4. Exposição OData (Service Definition e Binding)

**Service Definition**
Seleciona quais entidades (Consumption Views) serão expostas:
```abap
@EndUserText.label: 'Ordem de vendas'
define service ZUI_SALESORDER_ESCHOEPF {
  expose ZC_HEADEROV_ESCHOEPF as SalesOrder;
  expose ZC_ITEMOV_ESCHOEPF   as SalesOrderItem;
}
```

**Service Binding**

- Vincula a Service Definition a um protocolo (OData V4 recomendado)  
- Gera automaticamente o endpoint  
- Após ativar, a URL é fornecida pelo sistema

---

## ⚙️ 5. Definição de Comportamento (Behavior Definition & Implementation)

Necessária em **apps transacionais** (CRUD).  
Permite configurar:
- Operações permitidas (create, update, delete)
- Validações
- Determinações
- Controle de concorrência

**Exemplo Behavior Definition:**
```abap
CLASS zbp_i_productroot_eschoepf DEFINITION PUBLIC ABSTRACT FINAL FOR BEHAVIOR OF zi_productroot_eschoepf.
ENDCLASS.

CLASS zbp_i_productroot_eschoepf IMPLEMENTATION.
ENDCLASS.
```

```abap
// Início da implementação gerenciada de um behavior definition (definição de comportamento)
managed implementation in class zbp_i_productroot_eschoepf unique;

// Define que a implementação é *strict*, ou seja, segue regras mais rígidas de verificação (versão 2 das regras)
strict ( 2 );

// Define o *behavior* (comportamento) para a interface de dados ZI_PRODUCTROOT_ESCHOEPF
define behavior for ZI_PRODUCTROOT_ESCHOEPF alias Products

// Informa a tabela persistente que armazena os dados no banco de dados
persistent table ztb_product_esch

// Define o controle de bloqueio de registros como *master*, ou seja, ao editar um item, ele será bloqueado
lock master

// Controle de autorização no nível da instância (registro por registro)
authorization master ( instance )

// Define um campo para controle de concorrência otimista.
// Isso evita que duas pessoas atualizem o mesmo registro ao mesmo tempo sem saber.
// O campo 'LocalLastChangeAt' armazena a última data/hora de modificação do registro.
// Antes de salvar uma alteração, o sistema verifica se o valor ainda é o mesmo.
// Se alguém já tiver alterado o registro, a atualização é bloqueada para evitar perda de dados.
etag master LocalLastChangeAt

// Bloco de ações permitidas para essa entidade:
// Isso define que os registros podem ser criados, atualizados e deletados via o serviço RAP
{
  create;   // Permite criar novos registros
  update;   // Permite atualizar registros existentes
  delete;   // Permite excluir registros


  determination setProduct on modify { create; }
  validation    checkNetWeight on save { field NetWeight; create; update; }

  field ( numbering : managed, readonly ) ProductUuid;
  field ( readonly ) Product;
  field ( mandatory ) ProductDescription, Plant;

  mapping for ztb_product_esch
  {
        ProductUuid        = product_uuid;
        Product            = matnr;
        Plant              = werks;
        ProductDescription = maktx;
        NetWeight          = ntgew;
        WeightUnit         = gewei;
        LastChangeBy       = last_changed_by;
        LastChangeAt       = last_changed_at;
        LocalLastChangeAt  = local_last_changed_at;
  }
}
```

```abap
projection;
strict ( 2 );

define behavior for ZC_PRODUCTROOT_ESCHOEPF alias Products
{
  use create;
  use update;
  use delete;
}

```

🚀 Deploy e Teste no Fiori Launchpad
Com o serviço ativo e anotado corretamente, o app Fiori Elements pode ser consumido via Launchpad (FLP), App Manager, ou integrado em catálogos.  


## 📂 Estrutura genérica de arquivos no ABAP Development Tools (ADT)
```
/sap-fiori-elements
│
├── /Dictionary
│   └── /Database Tables         # Tabelas com campos persistentes que representam as entidades de negócio
│
├── /Core Data Services
│   ├── /Data Definitions        # CDS Views (Interface + Consumption)
│   │   ├── ZI_Entity            # Interface View: encapsula as tabelas e define associações (`association [0..*] to`)
│   │   └── ZC_Entity            # Consumption View: camada de projeção (projeta os dados para o consumo)
│   │
│   ├── /Behavior Definitions    # Comportamento das entidades RAP
│   │   └──  ZI_Entity           # Behavior Definition (CRUD, validações simples)
│   │
│   └── /Metadata Extensions     # Anotações UI (@UI)
│       └── ZC_Entity            # Define, por meio de anotações `@UI`, como os dados serão exibidos na interface (@UI.lineItem, @UI.FieldGroup, etc.)
│
└── /Business Services
    ├── /Service Definitions     # Exposição OData
    │   └── ZSD_EntityService    # Exposição das CDS Views como entidades OData
    │
    └── /Service Bindings        # Ativação do serviço
        └── ZSB_EntityService    # Configuração do protocolo (OData V2/V4) e tipo (UI)

```
