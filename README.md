# Visão Geral do Desenvolvimento RAP no Eclipse (ADT)
🔹 Interface CDS (ZI_...)
Define a estrutura de dados (campos, joins, associações).
Essa view normalmente é baseada em tabelas do sistema ou views reutilizáveis.

```abap
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Ordem de Vendas - Cabeçalho - Eduardo'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
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
