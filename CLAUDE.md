# Reconcilia

Proyecto de portafolio en C#/.NET que resuelve dos problemas de vendedores que despachan con varias transportadoras en Colombia:

1. **Conciliación de pagos contra-entrega:** cruza las liquidaciones de cada transportadora (Excel/CSV) contra los pedidos y detecta pagos faltantes, duplicados, montos distintos y comisiones mal descontadas.
2. **Monitoreo de envíos:** consulta el estado en varias transportadoras, lo unifica en un solo modelo y avisa cuando hay novedades.

Ambos comparten la entidad "Envío". Se construye por fases: primero la conciliación (Fase 1), luego el monitoreo (Fase 2).

## Contexto del desarrollador
Desarrollador junior con experiencia en C#, ASP.NET Core, EF Core y SQL Server, y con experiencia previa en logística de última milla. Responde siempre en español.

## Prioridades (en este orden)
1. Que resuelva un problema real, no un CRUD genérico. El valor está en las reglas de negocio.
2. Código limpio, mantenible y escalable.
3. Repositorio ordenado y fácil de entender para un reclutador.
4. Que se pueda demostrar sin depender de terceros: datos sintéticos con errores conocidos y transportadoras simuladas.

## Arquitectura
- Monolito modular con Clean Architecture. Módulos: Shipments (núcleo compartido), Reconciliation, Tracking.
- Capas por módulo: Domain (sin dependencias) → Application → Infrastructure → Api. Las dependencias solo apuntan hacia adentro.
- Comunicación entre módulos mediante eventos de dominio, nunca con referencias directas a sus internos.
- Strategy para los parsers de liquidación; un adapter por transportadora; reglas de conciliación como clases pequeñas que implementan `IReconciliationRule`.
- Importaciones idempotentes (hash del archivo). Auditoría de cambios de estado.
- Stack: .NET 10, minimal APIs, EF Core + PostgreSQL, FluentValidation, Serilog, Microsoft.Extensions.Http.Resilience, Hangfire, xUnit, Testcontainers, Bogus, WireMock.Net, Docker Compose. Sin librería de mediador: handlers simples inyectados por DI.

## Estructura del repositorio
```
reconcilia/
├── README.md
├── CLAUDE.md
├── docs/            (architecture.md, adr/, domain-glossary.md, progress.md)
├── src/
│   ├── BuildingBlocks/
│   ├── Shipments/       (Domain, Application, Infrastructure)
│   ├── Reconciliation/  (Domain, Application, Infrastructure)
│   ├── Tracking/        (Fase 2)
│   └── Reconcilia.Api/
├── tools/
│   ├── Reconcilia.DataGenerator/
│   └── Reconcilia.CarrierSimulator/
├── tests/           (Domain, Application, Integration, Architecture)
├── docker-compose.yml
├── Directory.Build.props
├── .editorconfig
└── .github/workflows/ci.yml
```
Nombres de proyectos: `Reconcilia.<Modulo>.<Capa>` (ej. `Reconcilia.Shipments.Domain`).

## Estándares de código
- Nullable habilitado, warnings como errores, analyzers y `.editorconfig` en el repo.
- Dominio rico: la lógica de negocio vive en entidades y value objects, no en servicios anémicos.
- Result pattern para errores esperados; excepciones solo para lo inesperado.
- Clases y métodos pequeños, nombres que expresan intención del negocio, sin comentarios que expliquen lo obvio.
- Sin sobreingeniería: no agregues abstracciones sin una razón concreta. Si propones un patrón, explica el problema que resuelve.
- Tests: reglas de dominio con tests unitarios de nombres descriptivos (`Escenario_ResultadoEsperado`), integración con Testcontainers, tests de arquitectura con NetArchTest.
- Antes de dar un paso por terminado: `dotnet build` y `dotnet test` en verde.

## Git
- Commits pequeños con Conventional Commits (`feat(shipments): add Shipment aggregate`).
- No hagas commit sin que yo lo apruebe. No toques la rama `main` con cambios sin revisar.

## Cómo trabajar conmigo
- Avanza por pasos pequeños y verificables, uno a la vez. No generes el proyecto completo de una sola vez.
- Antes de escribir código de una fase o paso nuevo, muéstrame el diseño en pocas líneas y espera mi aprobación.
- Si una decisión mía va en contra de las prioridades de este documento, dímelo con argumentos antes de implementarla.
- Cuando escribas código, explica brevemente el porqué de las decisiones de diseño no obvias, para que yo pueda defenderlas en una entrevista.
- Al empezar una sesión, lee `docs/progress.md`. Al terminar un paso, actualízalo con lo hecho y lo pendiente.
- Sé conciso y directo.