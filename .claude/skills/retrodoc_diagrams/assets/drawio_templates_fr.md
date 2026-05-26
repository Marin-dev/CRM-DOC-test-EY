# Draw.io Templates FR

## Format de sortie attendu
- Fichier `.drawio` au format XML mxGraphModel
- Importable directement dans desktop.draw.io ou app.diagrams.net
- Polices web-safe (Helvetica/Arial)
- Légende en bas, palette sobre

## Squelette XML — Architecture "exec-friendly"

À copier dans `docs/retrodoc/diagrams/drawio_architecture.drawio` puis adapter aux composants réellement détectés.

```xml
<mxfile host="app.diagrams.net">
  <diagram name="Architecture" id="arch-main">
    <mxGraphModel dx="1200" dy="800" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1169" pageHeight="826" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />

        <!-- TITRE -->
        <mxCell id="title" value="Architecture — &lt;Nom application&gt;" style="text;html=1;align=center;verticalAlign=middle;fontSize=18;fontStyle=1" vertex="1" parent="1">
          <mxGeometry x="400" y="20" width="400" height="30" as="geometry" />
        </mxCell>

        <!-- UTILISATEUR -->
        <mxCell id="user" value="Utilisateur" style="shape=actor;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf" vertex="1" parent="1">
          <mxGeometry x="60" y="120" width="40" height="60" as="geometry" />
        </mxCell>

        <!-- FRONTEND -->
        <mxCell id="web" value="Web&#xa;(React/Vite)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366" vertex="1" parent="1">
          <mxGeometry x="180" y="120" width="140" height="60" as="geometry" />
        </mxCell>

        <!-- BACKEND -->
        <mxCell id="api" value="API&#xa;(Node/Express)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656" vertex="1" parent="1">
          <mxGeometry x="400" y="120" width="140" height="60" as="geometry" />
        </mxCell>

        <mxCell id="worker" value="Worker&#xa;(Jobs async)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656" vertex="1" parent="1">
          <mxGeometry x="400" y="220" width="140" height="60" as="geometry" />
        </mxCell>

        <!-- DB -->
        <mxCell id="db" value="Postgres" style="shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;size=15;fillColor=#f8cecc;strokeColor=#b85450" vertex="1" parent="1">
          <mxGeometry x="640" y="120" width="100" height="60" as="geometry" />
        </mxCell>

        <mxCell id="cache" value="Redis" style="shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;size=15;fillColor=#f8cecc;strokeColor=#b85450" vertex="1" parent="1">
          <mxGeometry x="640" y="220" width="100" height="60" as="geometry" />
        </mxCell>

        <!-- INFRA -->
        <mxCell id="cdn" value="CDN" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6" vertex="1" parent="1">
          <mxGeometry x="180" y="40" width="140" height="40" as="geometry" />
        </mxCell>

        <mxCell id="lb" value="Load Balancer" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6" vertex="1" parent="1">
          <mxGeometry x="400" y="40" width="140" height="40" as="geometry" />
        </mxCell>

        <!-- TIERS -->
        <mxCell id="stripe" value="Stripe" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;dashed=1" vertex="1" parent="1">
          <mxGeometry x="820" y="120" width="120" height="40" as="geometry" />
        </mxCell>

        <mxCell id="idp" value="IdP (OIDC)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;dashed=1" vertex="1" parent="1">
          <mxGeometry x="820" y="180" width="120" height="40" as="geometry" />
        </mxCell>

        <!-- FLUX -->
        <mxCell id="e1" value="HTTPS" style="endArrow=classic;html=1" edge="1" parent="1" source="user" target="web">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e2" value="REST /v1 (JSON)" style="endArrow=classic;html=1" edge="1" parent="1" source="web" target="api">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e3" value="SQL" style="endArrow=classic;html=1" edge="1" parent="1" source="api" target="db">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e4" value="Cache / Queue" style="endArrow=classic;html=1" edge="1" parent="1" source="api" target="cache">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e5" value="Event" style="endArrow=classic;html=1;dashed=1" edge="1" parent="1" source="api" target="worker">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e6" value="API + Webhook" style="endArrow=classic;html=1" edge="1" parent="1" source="api" target="stripe">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>
        <mxCell id="e7" value="OIDC" style="endArrow=classic;html=1" edge="1" parent="1" source="api" target="idp">
          <mxGeometry relative="1" as="geometry" />
        </mxCell>

        <!-- LÉGENDE -->
        <mxCell id="legend-title" value="Légende" style="text;html=1;fontStyle=1" vertex="1" parent="1">
          <mxGeometry x="60" y="380" width="120" height="20" as="geometry" />
        </mxCell>
        <mxCell id="legend-front" value="Frontend" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366" vertex="1" parent="1">
          <mxGeometry x="60" y="410" width="100" height="30" as="geometry" />
        </mxCell>
        <mxCell id="legend-back" value="Backend" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656" vertex="1" parent="1">
          <mxGeometry x="170" y="410" width="100" height="30" as="geometry" />
        </mxCell>
        <mxCell id="legend-db" value="Data" style="shape=cylinder3;whiteSpace=wrap;html=1;boundedLbl=1;backgroundOutline=1;size=15;fillColor=#f8cecc;strokeColor=#b85450" vertex="1" parent="1">
          <mxGeometry x="280" y="410" width="80" height="30" as="geometry" />
        </mxCell>
        <mxCell id="legend-infra" value="Infra" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e1d5e7;strokeColor=#9673a6" vertex="1" parent="1">
          <mxGeometry x="370" y="410" width="80" height="30" as="geometry" />
        </mxCell>
        <mxCell id="legend-ext" value="Tiers externe" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;dashed=1" vertex="1" parent="1">
          <mxGeometry x="460" y="410" width="120" height="30" as="geometry" />
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

## Règles de personnalisation
- **Couleurs** : conserver la palette par catégorie (front/back/data/infra/tiers).
- **Composants `INCONNU`** : utiliser un fond gris clair `#e0e0e0` et le label `INCONNU — <hypothèse>`.
- **Flèches** : libeller systématiquement (REST, event, SQL, batch, webhook…).
- **Disposition** : utilisateur à gauche → frontend → backend → data → tiers à droite.
- **Taille** : viser ≤ 12 nœuds principaux. Si trop, créer un deuxième diagramme (zoom sur un sous-système).
