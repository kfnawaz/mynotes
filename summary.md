**SPI stands for Service Provider Interface.**

In Java, an SPI is a set of interfaces or contracts that allows different implementations (service providers) to plug into a common framework.

### **In your DataCompass Crawlers project**

The **Connector SPI** defines a standard contract that every metadata crawler must implement.

For example:

- **Databricks Connector** — Crawls catalogs, schemas, tables, and lineage metadata.
- **Snowflake Connector** — Crawls Snowflake metadata.
- **Tableau Connector** — Crawls Tableau workbooks, dashboards, and data sources.

Instead of developing separate crawler engines for each platform, you build **one common crawler engine** and multiple connectors that implement the same SPI.

### **Java example**

```
public interface CrawlerConnector {
    String getConnectorType();

    void validateConnection();

    CrawlResult crawl(CrawlContext context);
}
```

Each connector implements this interface:

```
public class DatabricksConnector
        implements CrawlerConnector {

    @Override
    public String getConnectorType() {
        return "DATABRICKS";
    }

    @Override
    public void validateConnection() {
        // Validate Databricks connectivity
    }

    @Override
    public CrawlResult crawl(CrawlContext context) {
        // Extract Databricks metadata
        return new CrawlResult();
    }
}
```

### **How it works**

**Common Crawler Engine (Java)**

Scheduling, execution, run state, logging, errors

**Connector SPI (Java Interface)**

Standard contract every connector follows

Databricks

Connector

Snowflake

Connector

Tableau

Connector

**Why it matters:** Once the SPI is defined, you can add a new connector without redesigning the crawler engine.

For your SDD approach, the Connector SPI should be an **approved, versioned architecture contract** before Claude starts building individual connectors. It should define more than method names: supported capabilities, lifecycle, pagination, retries, errors, and metadata output.

In simple terms: **The crawler engine controls how crawling runs; the connector SPI defines how each platform-specific crawler participates.**