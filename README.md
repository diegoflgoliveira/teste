# Release v${{ needs.build.outputs.version }}
          
          ---
          
          ## Build Information
          
          | Atributo | Valor |
          |----------|-------|
          | **Versão** | `${{ needs.build.outputs.version }}` |
          | **Commit SHA** | [`${GITHUB_SHA:0:7}`](https://github.com/${{ github.repository }}/commit/${{ github.sha }}) |
          | **Branch** | `${{ github.ref_name }}` |
          | **Build Number** | `#${{ github.run_number }}` |
          | **Executor** | @${{ github.actor }} |
          | **Workflow** | [`${{ github.workflow }}`](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}) |
          | **Data/Hora** | `$(date -u +"%Y-%m-%d %H:%M:%S UTC")` |
          
          END_SUMMARY
          
          if [ -n "${{ inputs.version-override }}" ]; then
            echo "| **Estratégia** | Manual Override |" >> $GITHUB_STEP_SUMMARY
          elif [ "${{ inputs.bump-type }}" != "none" ]; then
            echo "| **Estratégia** | Auto-increment (\`${{ inputs.bump-type }}\`) |" >> $GITHUB_STEP_SUMMARY
          else
            echo "| **Estratégia** | package.json |" >> $GITHUB_STEP_SUMMARY
          fi
          
          cat >> $GITHUB_STEP_SUMMARY << 'END_SUMMARY'
          
          ---
          
          ## Docker Images
          
          ### Imagens Publicadas
          
          ```bash
          # Versão específica (recomendado para produção)
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:${{ needs.build.outputs.version }}
          
          # Latest (desenvolvimento)
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:latest
          
          # Homologação
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:homolog
          
          # SHA específico
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:sha-${GITHUB_SHA:0:7}
          
          # Build number
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:build-${{ github.run_number }}
          ```
          
          ### Tags Disponíveis
          
          | Tag | Uso Recomendado | Estabilidade |
          |-----|----------------|--------------|
          | `${{ needs.build.outputs.version }}` | Produção | Stable |
          | `latest` | Desenvolvimento | Rolling |
          | `homolog` | Homologação | Testing |
          | `sha-${GITHUB_SHA:0:7}` | Debug | Immutable |
          | `build-${{ github.run_number }}` | Rastreamento | Immutable |
          
          ---
          
          ## Quality Gates
          
          | Gate | Status | Detalhes |
          |------|--------|----------|
          | **Build** | ✅ Passed | TypeScript compilation successful |
          | **Unit Tests** | ✅ Passed | All unit tests passed |
          | **E2E Tests** | ✅ Passed | Integration tests validated |
          | **Security Scan** | ✅ Passed | Trivy vulnerability scan completed |
          | **Lint** | ✅ Passed | Hadolint Docker analysis |
          
          ---
          
          ## Test Results
          
          ### Testes Unitários
          - [X] Executados com sucesso
          - [X] Coverage disponível nos artifacts
          
          ### Testes E2E
          - [X] Todos os cenários validados
          - [X] Infraestrutura Docker provisionada automaticamente
          - [X] Testes gerenciam seus próprios dados (sem overhead de seed)
          
          ---
          
          ## Security & Compliance
          
          - Trivy vulnerability scan: **Completed**
          - SARIF report uploaded to GitHub Security
          - Docker image signed and verified
          - No critical vulnerabilities detected
          
          ---
          
          ## Deployment
          
          ### Docker Compose
          
          ```yaml
          version: '3.8'
          services:
            api:
              image: ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:${{ needs.build.outputs.version }}
              ports:
                - "3000:3000"
              environment:
                NODE_ENV: production
                DATABASE_URL: postgresql://...
                MONGO_INITDB_URI: mongodb://...
                REDIS_URL: redis://...
          ```
          
          ### Kubernetes
          
          ```bash
          kubectl set image deployment/helpme-api \
            api=${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:${{ needs.build.outputs.version }}
          ```
          
          ---
          
          ## Resources
          
          - [Documentação da API](https://github.com/${{ github.repository }})
          - [Docker Hub](https://hub.docker.com/r/${{ secrets.DOCKERHUB_USERNAME }}/helpme-api)
          - [Swagger/OpenAPI](http://localhost:3000/api-docs)
          - [Grafana Dashboards](https://github.com/${{ github.repository }}/tree/main/api/painel-analitico/grafana/dashboards)
          
          ---
          
          ## Next Steps
          
          1. **Produção**: Deploy da imagem `${{ needs.build.outputs.version }}` no ambiente de produção
          2. **Monitoramento**: Verificar métricas no Grafana após deploy
          3. **Rollback**: Manter imagem anterior disponível para rollback rápido
          4. **Documentação**: Atualizar changelog com as mudanças desta versão
          
          ---
          
          <div align="center">
          
          ** Pipeline Executada com Sucesso! **
          
          [![View Workflow](https://img.shields.io/badge/View-Workflow-blue?style=for-the-badge&logo=github)](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }})
          [![Docker Hub](https://img.shields.io/badge/Docker-Hub-2496ED?style=for-the-badge&logo=docker)](https://hub.docker.com/r/${{ secrets.DOCKERHUB_USERNAME }}/helpme-api)
          
          </div>
          END_SUMMARY

  github-release:
    name: Create GitHub Release
    runs-on: ubuntu-latest
    needs: [build, release]
    if: ${{ inputs.create-tag == true }}
    environment: homologacao
    permissions:
      contents: write
    
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Gerar Release Notes
        id: release_notes
        run: |
          VERSION="${{ needs.build.outputs.version }}"
          
          PREVIOUS_TAG=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")
          
          if [ -n "$PREVIOUS_TAG" ]; then
            COMMITS=$(git log ${PREVIOUS_TAG}..HEAD --pretty=format:"- %s ([%h](https://github.com/${{ github.repository }}/commit/%H))" --no-merges)
          else
            COMMITS=$(git log --pretty=format:"- %s ([%h](https://github.com/${{ github.repository }}/commit/%H))" --no-merges -n 20)
          fi
          
          cat > release_notes.md << 'RELEASE_END'
          ## Help-Me API vVERSION_PLACEHOLDER
          
          Today, we are excited to share the **vVERSION_PLACEHOLDER** stable release
          
          **Star this repo** for notifications about new releases, bug fixes & features!
          
          ---
          
          ## Installation
          
          ### Docker (Recommended)
          
          ```bash
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:VERSION_PLACEHOLDER
          
          docker run -d -p 3000:3000 \
            -e DATABASE_URL="postgresql://user:pass@localhost:5432/helpme" \
            -e MONGO_INITDB_URI="mongodb://user:pass@localhost:27017/helpme-mongo" \
            -e REDIS_URL="redis://localhost:6379" \
            -e JWT_SECRET="your-secret-here" \
            -e JWT_REFRESH_SECRET="your-refresh-secret-here" \
            ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:VERSION_PLACEHOLDER
          ```
          
          ### Docker Compose
          
          ```yaml
          version: '3.8'
          services:
            api:
              image: ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:VERSION_PLACEHOLDER
              ports:
                - "3000:3000"
              environment:
                NODE_ENV: production
                DATABASE_URL: postgresql://user:pass@postgres:5432/helpme
                MONGO_INITDB_URI: mongodb://user:pass@mongodb:27017/helpme-mongo
                REDIS_URL: redis://redis:6379
                KAFKA_BROKER_URL: kafka:9093
                JWT_SECRET: your-secret-here
                JWT_REFRESH_SECRET: your-refresh-secret-here
          ```
          
          ### Kubernetes
          
          ```bash
          kubectl set image deployment/helpme-api \
            api=${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:VERSION_PLACEHOLDER
          ```
          
          ---
          
          ## What's New
          
          RELEASE_END
          
          echo "" >> release_notes.md
          echo "$COMMITS" >> release_notes.md
          echo "" >> release_notes.md
          
          cat >> release_notes.md << 'RELEASE_END'
          
          ---
          
          ## Technical Details
          
          | Attribute | Value |
          |-----------|-------|
          | **Version** | `VERSION_PLACEHOLDER` |
          | **Build** | #${{ github.run_number }} |
          | **Commit** | [`COMMIT_PLACEHOLDER`](${{ github.server_url }}/${{ github.repository }}/commit/${{ github.sha }}) |
          | **Node.js** | 22.x |
          | **TypeScript** | 5.9.x |
          | **Prisma** | 7.x |
          | **Docker Base** | node:22-alpine |
          
          ---
          
          ## Quality Metrics
          
          - [X] **Build**: Passed
          - [X] **Unit Tests**: All tests passed
          - [X] **E2E Tests**: Integration validated
          - [X] **Security Scan**: No critical vulnerabilities
          - [X] **Docker Build**: Multi-stage optimized
          
          ---
          
          ## Docker Images
          
          ### Available Tags
          
          | Tag | Purpose | Stability |
          |-----|---------|-----------|
          | `VERSION_PLACEHOLDER` | Production | Stable |
          | `latest` | Development | Rolling |
          | `homolog` | Staging | Testing |
          | `sha-COMMIT_PLACEHOLDER` | Debug | Immutable |
          
          ```bash
          # Production (recommended)
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:VERSION_PLACEHOLDER
          
          # Latest
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:latest
          
          # Specific commit
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/helpme-api:sha-COMMIT_PLACEHOLDER
          ```
          
          ---
          
          ## Documentation
          
          - [API Documentation](https://github.com/${{ github.repository }})
          - [Swagger/OpenAPI](http://localhost:3000/api-docs)
          - [Docker Hub](https://hub.docker.com/r/${{ secrets.DOCKERHUB_USERNAME }}/helpme-api)
          - [Grafana Dashboards](https://github.com/${{ github.repository }}/tree/main/api/painel-analitico/grafana/dashboards)
          - [Kubernetes Manifests](https://github.com/${{ github.repository }}/tree/main/api/k8s)
          
          ---
          
          ## Support
          
          - [Report a Bug](https://github.com/${{ github.repository }}/issues/new?labels=bug)
          - [Request a Feature](https://github.com/${{ github.repository }}/issues/new?labels=enhancement)
          - [Discussions](https://github.com/${{ github.repository }}/discussions)
          
          ---
          
          ## Full Changelog
          
          RELEASE_END
          
          if [ -n "$PREVIOUS_TAG" ]; then
            echo "**[$PREVIOUS_TAG...v$VERSION](${{ github.server_url }}/${{ github.repository }}/compare/$PREVIOUS_TAG...v$VERSION)**" >> release_notes.md
          else
            echo "**[Initial Release](https://github.com/${{ github.repository }}/commits/v$VERSION)**" >> release_notes.md
          fi
          
          cat >> release_notes.md << 'RELEASE_END'
          
          ---
          
          <div align="center">
          
          **Made with by the Help-Me Team**
          
          [![Docker Pulls](https://img.shields.io/docker/pulls/${{ secrets.DOCKERHUB_USERNAME }}/helpme-api?style=flat-square)](https://hub.docker.com/r/${{ secrets.DOCKERHUB_USERNAME }}/helpme-api)
          [![GitHub Stars](https://img.shields.io/github/stars/${{ github.repository }}?style=flat-square)](https://github.com/${{ github.repository }}/stargazers)
          [![License](https://img.shields.io/github/license/${{ github.repository }}?style=flat-square)](https://github.com/${{ github.repository }}/blob/main/LICENSE)
          
          </div>
          RELEASE_END
