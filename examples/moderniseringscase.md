# Moderniseringscase: Nordisk Handel AB

Nordisk Handel AB är en fiktiv organisation med en affärskritisk Java EE-plattform. Plattformen använder traditionell applikationsserver, Oracle Database och IBM MQ.

## Nuläge
- Java EE-applikationer i traditionell runtime.
- Oracle som central databasplattform.
- IBM MQ som integrationsnav.
- Begränsad automatisering av deployment.
- Drift och utveckling är organisatoriskt separerade.

## Målbild
- OpenShift som standardplattform för applikationsdrift.
- Podman för lokal containerbyggnad och verifiering.
- Stegvis modernisering av data- och integrationslager.
- Elasticsearch för sök, logg- och analysnära användningsfall.
- Ceph för block-, fil- och objektlagring där det passar.
- Styrda teknikval dokumenterade med ADR:er.
