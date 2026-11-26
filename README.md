# Deplyn API

## Het eerste Flyway-schema exporteren

Zodra de eerste JPA-entities klaar zijn, kun je Hibernate de SQL voor de eerste migratie laten aanmaken.

1. Maak in de lokale PostgreSQL-container een lege wegwerpdatabase aan.

   ```bash
   docker exec deplyn-database createdb -U postgres deplyn_schema_export
   ```

2. Start de applicatie eenmalig met het profiel `local` en de volgende omgevingsvariabelen. Gebruik je lokale databasewachtwoord voor `SPRING_DATASOURCE_PASSWORD`. Controleer de URL voordat je start, want Hibernate kan met `create` bestaande tabellen verwijderen.

   ```dotenv
   SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:3307/deplyn_schema_export
   SPRING_DATASOURCE_USERNAME=postgres
   SPRING_FLYWAY_ENABLED=false
   SPRING_JPA_HIBERNATE_DDL_AUTO=create
   ```

3. Exporteer het schema vanuit de projectmap. Het bestand `schema-export.sql` wordt op je eigen machine geschreven.

   ```bash
   docker exec deplyn-database pg_dump -U postgres -d deplyn_schema_export \
     --schema=public --schema-only --no-owner --no-privileges > schema-export.sql
   ```

4. Controleer tabellen, constraints en indexen. Haal eventuele `psql`-commando's zoals `\restrict` en `\unrestrict` uit de export en zet de gecontroleerde SQL in `src/main/resources/db/migration/V1__initial_schema.sql`.
5. Test `V1` tegen een andere lege PostgreSQL-database met `SPRING_FLYWAY_ENABLED=true` en `SPRING_JPA_HIBERNATE_DDL_AUTO=validate`. Controleer of versie `1` als geslaagd in `flyway_schema_history` staat. Houd voor normaal gebruik Flyway aan en Hibernate op `validate`.
