# dbt-dim-over-datavault

In a star schema generated, dimensions are based on entities, and facts are based on transactional links. Relationships are not mapped directly but are used to connect entities for snowflake-style usage or for denormalizing into flattened facts.

For dimension tables, we provide two flavors: with and without history tracking. You get to choose between SCD1 and SCD2 for individual dimensions (of course, our query API handles that for users automatically). In order to efficiently generate dimension tables, we take advantage of point-in-time tables (PIT). By periodically and incrementally generating PIT, our star schema dimension tables can simply be views defined on PITs together with their base hubs/satellites. This is key to our users as they get SCD2 for free, if and when needed.

For fact tables, two options are available. For simple cases, say a transaction line item data persisted in a transactional link, the corresponding fact table is simply incrementally synchronized from the link table. The line items, products, and users referenced in the data all become the fact table’s foreign references to different dimension tables. This is a simple 1-to-1 mapping from data vault to the generated star schema.

For more complicated cases, usually, a chain of links and hubs need to be joined first to enrich the transactional links with the required dimensions. In this case, we allow specifying the “join chain” and the intended dimensions. Essentially, it creates a “denormalized” fact table. An example would be IoT data, which usually comes with just the sensor ID. Of course, we can go with snowflake, but we are really more star guys. By specifying the “join chain” from the sensor ID to its belonging equipment, maybe also further from equipment to the belonging organization, location, and work order, it is very easy to populate a required fact table to serve different business needs.

Remember since the data vault is the source of truth, any time a new business requirement comes in, we can either update an existing star schema suitable for the purpose, or simply drop-recreate one, or just go creating a new star schema. Either way, all other star schemas are not affected.

