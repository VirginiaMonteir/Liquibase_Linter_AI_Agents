# Interface de Aprovação para Grupo AD (Administradores de Dados)

## 1. Visão Geral

Este documento detalha o design e implementação da interface de aprovação destinada ao grupo AD (Administradores de Dados) para revisão e decisão sobre exceções `linter-ignore-rule` de alta severidade em changesets do Liquibase.

## 2. Requisitos da Interface

### 2.1 Requisitos Funcionais
1. **Visualização de Exceções**: Mostrar todas as exceções pendentes de aprovação
2. **Detalhamento de Exceções**: Exibir informações contextuais detalhadas
3. **Tomada de Decisão**: Permitir aprovar, rejeitar ou solicitar alterações
4. **Justificativa**: Capturar justificativas para todas as decisões
5. **Histórico de Aprovações**: Visualizar histórico de decisões anteriores
6. **Notificações**: Receber alertas sobre exceções críticas pendentes

### 2.2 Requisitos Não-Funcionais
1. **Segurança**: Acesso restrito apenas ao grupo AD
2. **Usabilidade**: Interface intuitiva e fácil de navegar
3. **Responsividade**: Funcionar bem em dispositivos móveis e desktop
4. **Performance**: Carregar rapidamente mesmo com muitas exceções
5. **Auditabilidade**: Registrar todas as ações realizadas

## 3. Design da Interface

### 3.1 Layout Geral
```
+-------------------------------------------------------------+
| LOGO EMPRESA | Painel de Aprovações - Administradores de Dados |
+-------------------------------------------------------------+
| FILTROS | BUSCA | NOTIFICAÇÕES                              |
+-------------------------------------------------------------+
| RESUMO DASHBOARD                                            |
| +----------+ +----------+ +----------+ +----------+         |
| |Pendentes | |Aprovadas | |Rejeitadas| |Timeout   |         |
| |    5     | |   23     | |    2     | |    1     |         |
| +----------+ +----------+ +----------+ +----------+         |
+-------------------------------------------------------------+
| LISTA DE EXCEÇÕES PENDENTES                                 |
| +---------------------------------------------------------+ |
| | [ALTA] changeset: dev.junior:migration-drop-temp-tables | |
| | Regra: no-drop-table | Severidade: Alta                 | |
| | Arquivo: db/changelog/v1.2.0/001-drop-temp-tables.xml   | |
| | Justificativa Dev: Remoção necessária de tabelas...     | |
| |                                                         | |
| | [ACÕES] [DETALHES] [APROVAR] [REJEITAR]                 | |
| +---------------------------------------------------------+ |
|                                                             |
| +---------------------------------------------------------+ |
| | [CRÍTICA] changeset: dba.admin:security-fix-123         | |
| | Regra: forbid-sale-schema | Severidade: Crítica         | |
| | Arquivo: security/fixes/security-patch.xml              | |
| | Justificativa Dev: Correção de vulnerabilidade crítica  | |
| |                                                         | |
| | [ACÕES] [DETALHES] [APROVAR] [REJEITAR]                 | |
| +---------------------------------------------------------+ |
+-------------------------------------------------------------+
| PAGINAÇÃO | EXPORTAR | AJUDA                               |
+-------------------------------------------------------------+
```

### 3.2 Página de Detalhamento de Exceção

#### 3.2.1 Seção de Informações do Changeset
```
INFORMAÇÕES DO CHANGESET
┌─────────────────────────────────────────────────────────────┐
│ Autor:           dev.junior                                │
│ ID:              migration-drop-temp-tables                │
│ Arquivo:         db/changelog/v1.2.0/001-drop-temp-tables.xml│
│ Data de Criação: 2026-02-01 14:30:22                       │
│ Hash:            a1b2c3d4-e5f6-7890-abcd-ef1234567890      │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.2 Seção de Informações da Exceção
```
DETALHES DA EXCEÇÃO
┌─────────────────────────────────────────────────────────────┐
│ Regra Ignorada:    no-drop-table                           │
│ Categoria:         Segurança                               │
│ Severidade Base:   Alta                                    │
│ Severidade Final:  Alta                                    │
│ Linha:             15                                      │
│ Ambiente:          Produção                                │
│ Status:            Pendente de Aprovação                   │
└─────────────────────────────────────────────────────────────┘
```

#### 3.2.3 Seção de Contexto do Código
```sql
-- Conteúdo do Changeset (com destaque para a exceção)
12 --changeset dev.junior:migration-drop-temp-tables
13 --linter-ignore-rule:no-drop-table
14 -- Justificativa: Remoção necessária de tabelas temporárias criadas durante processos batch
15 DROP TABLE temp_batch_processing_2023_01;
16 DROP TABLE temp_user_sessions_cleanup;
```

#### 3.2.4 Seção de Histórico do Autor
```
HISTÓRICO DO AUTOR
┌─────────────────────────────────────────────────────────────┐
│ Total de Exceções Solicitadas: 8                           │
│ Taxa de Aprovação: 75%                                     │
│ Última Exceção Aprovada: 2026-01-28                        │
│ Padrões de Solicitação: Comum em operações de limpeza      │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 Modal de Tomada de Decisão
```
[APROVAR EXCEÇÃO]

Justificativa para a Decisão:
[TextArea com altura suficiente para texto detalhado]

[X] Marcar como padrão confiável (não solicitar aprovação para casos similares)
[ ] Notificar autor sobre a decisão
[ ] Encaminhar cópia para revisão de segurança

+----------------+ +------------------+ +--------------+
| APROVAR        | | REJEITAR         | | CANCELAR     |
+----------------+ +------------------+ +--------------+
```

## 4. Implementação Técnica

### 4.1 Estrutura de Componentes Frontend
```
/src/web/components/approval-interface/
├── ApprovalDashboard.vue          # Dashboard principal
├── ExceptionList.vue              # Lista de exceções
├── ExceptionDetail.vue            # Detalhe da exceção
├── DecisionModal.vue              # Modal de tomada de decisão
├── SummaryCards.vue               # Cards de resumo
├── ExceptionFilter.vue            # Componente de filtros
└── NotificationBell.vue           # Ícone de notificações

/src/web/components/shared/
├── CodeSnippet.vue                # Visualização de código
├── SeverityBadge.vue              # Badge de severidade
├── AuthorHistory.vue              # Histórico do autor
└── ActionButtons.vue              # Botões de ação
```

### 4.2 Componente Principal - Dashboard
```vue
<!-- /src/web/components/approval-interface/ApprovalDashboard.vue -->
<template>
  <div class="approval-dashboard">
    <!-- Header -->
    <header class="dashboard-header">
      <h1>Painel de Aprovações - Administradores de Dados</h1>
      <div class="header-controls">
        <NotificationBell />
        <SearchBox @search="handleSearch" />
      </div>
    </header>

    <!-- Summary Cards -->
    <SummaryCards 
      :stats="dashboardStats"
      @filter-change="handleFilterChange"
    />

    <!-- Filters -->
    <ExceptionFilter 
      :filters="activeFilters"
      @update="updateFilters"
    />

    <!-- Exception List -->
    <ExceptionList 
      :exceptions="filteredExceptions"
      :loading="isLoading"
      @refresh="loadExceptions"
      @view-detail="showExceptionDetail"
    />

    <!-- Detail Modal (condicional) -->
    <ExceptionDetail 
      v-if="selectedException"
      :exception="selectedException"
      @close="closeDetail"
      @decision-made="handleDecisionMade"
    />
  </div>
</template>

<script>
import ApprovalService from '@/services/ApprovalService';
import { mapGetters, mapActions } from 'vuex';

export default {
  name: 'ApprovalDashboard',
  data() {
    return {
      isLoading: false,
      exceptions: [],
      filteredExceptions: [],
      selectedException: null,
      activeFilters: {
        severity: 'all',
        status: 'pending',
        environment: 'all'
      }
    };
  },
  computed: {
    ...mapGetters(['dashboardStats']),
    ...mapGetters(['userPermissions'])
  },
  async mounted() {
    await this.loadExceptions();
    await this.loadDashboardStats();
  },
  methods: {
    ...mapActions(['loadDashboardStats']),
    
    async loadExceptions() {
      this.isLoading = true;
      try {
        this.exceptions = await ApprovalService.getPendingApprovals();
        this.applyFilters();
      } catch (error) {
        console.error('Erro ao carregar exceções:', error);
      } finally {
        this.isLoading = false;
      }
    },

    applyFilters() {
      this.filteredExceptions = this.exceptions.filter(exception => {
        // Filtrar por severidade
        if (this.activeFilters.severity !== 'all' && 
            exception.calculatedSeverity !== this.activeFilters.severity) {
          return false;
        }

        // Filtrar por status
        if (this.activeFilters.status !== 'all' && 
            exception.status !== this.activeFilters.status) {
          return false;
        }

        // Filtrar por ambiente
        if (this.activeFilters.environment !== 'all' && 
            exception.environment !== this.activeFilters.environment) {
          return false;
        }

        return true;
      });
    },

    handleFilterChange(newFilters) {
      this.activeFilters = { ...this.activeFilters, ...newFilters };
      this.applyFilters();
    },

    showExceptionDetail(exception) {
      this.selectedException = exception;
    },

    closeDetail() {
      this.selectedException = null;
    },

    async handleDecisionMade(decisionData) {
      try {
        await ApprovalService.recordDecision(decisionData);
        this.closeDetail();
        await this.loadExceptions(); // Recarregar lista
        this.$toast.success('Decisão registrada com sucesso');
      } catch (error) {
        this.$toast.error('Erro ao registrar decisão: ' + error.message);
      }
    }
  }
};
</script>
```

### 4.3 Componente de Detalhe de Exceção
```vue
<!-- /src/web/components/approval-interface/ExceptionDetail.vue -->
<template>
  <div class="modal-overlay" @click="close">
    <div class="exception-detail-modal" @click.stop>
      <header class="modal-header">
        <h2>Detalhes da Exceção</h2>
        <button class="close-button" @click="close">×</button>
      </header>

      <div class="modal-content">
        <!-- Informações do Changeset -->
        <section class="changeset-info">
          <h3>Informações do Changeset</h3>
          <div class="info-grid">
            <InfoItem label="Autor" :value="exception.changeset.author" />
            <InfoItem label="ID" :value="exception.changeset.id" />
            <InfoItem label="Arquivo" :value="exception.fileName" />
            <InfoItem label="Data de Criação" :value="formatDate(exception.detectedAt)" />
          </div>
        </section>

        <!-- Detalhes da Exceção -->
        <section class="exception-details">
          <h3>Detalhes da Exceção</h3>
          <div class="details-grid">
            <InfoItem label="Regra Ignorada" :value="exception.ignoredRule.name" />
            <InfoItem label="Categoria" :value="exception.ignoredRule.category" />
            <InfoItem label="Severidade Base" :value="exception.ignoredRule.baseSeverity" />
            <InfoItem label="Severidade Final" :value="exception.calculatedSeverity" />
            <InfoItem label="Linha" :value="exception.lineNumber.toString()" />
            <InfoItem label="Ambiente" :value="exception.environment" />
          </div>
          
          <div class="severity-display">
            <SeverityBadge :severity="exception.calculatedSeverity" />
          </div>
        </section>

        <!-- Justificativa do Desenvolvedor -->
        <section class="developer-justification">
          <h3>Justificativa do Desenvolvedor</h3>
          <blockquote>{{ exception.justification || 'Nenhuma justificativa fornecida' }}</blockquote>
        </section>

        <!-- Contexto do Código -->
        <section class="code-context">
          <h3>Contexto do Código</h3>
          <CodeSnippet 
            :code="codeContext" 
            :highlight-line="exception.lineNumber"
            language="sql"
          />
        </section>

        <!-- Histórico do Autor -->
        <section class="author-history" v-if="authorHistory">
          <h3>Histórico do Autor</h3>
          <AuthorHistory :history="authorHistory" />
        </section>
      </div>

      <!-- Área de Decisão -->
      <footer class="decision-area">
        <div class="decision-form">
          <textarea 
            v-model="decisionJustification"
            placeholder="Justificativa para a decisão..."
            rows="4"
          ></textarea>
          
          <div class="decision-options">
            <label>
              <input type="checkbox" v-model="markAsTrusted">
              Marcar como padrão confiável
            </label>
            <label>
              <input type="checkbox" v-model="notifyAuthor">
              Notificar autor sobre a decisão
            </label>
          </div>
          
          <div class="decision-buttons">
            <button class="btn btn-success" @click="makeDecision('approve')">
              Aprovar
            </button>
            <button class="btn btn-danger" @click="makeDecision('reject')">
              Rejeitar
            </button>
            <button class="btn btn-secondary" @click="close">
              Cancelar
            </button>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
import ApprovalService from '@/services/ApprovalService';

export default {
  name: 'ExceptionDetail',
  props: {
    exception: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      decisionJustification: '',
      markAsTrusted: false,
      notifyAuthor: true,
      codeContext: '',
      authorHistory: null
    };
  },
  async mounted() {
    await this.loadCodeContext();
    await this.loadAuthorHistory();
  },
  methods: {
    async loadCodeContext() {
      try {
        const context = await ApprovalService.getCodeContext(
          this.exception.fileName,
          this.exception.lineNumber
        );
        this.codeContext = context;
      } catch (error) {
        console.error('Erro ao carregar contexto do código:', error);
        this.codeContext = 'Erro ao carregar contexto do código';
      }
    },

    async loadAuthorHistory() {
      try {
        this.authorHistory = await ApprovalService.getAuthorHistory(
          this.exception.changeset.author
        );
      } catch (error) {
        console.error('Erro ao carregar histórico do autor:', error);
      }
    },

    formatDate(dateString) {
      return new Date(dateString).toLocaleString('pt-BR');
    },

    close() {
      this.$emit('close');
    },

    async makeDecision(decision) {
      if (!this.decisionJustification.trim()) {
        this.$toast.warning('Por favor, forneça uma justificativa para a decisão');
        return;
      }

      const decisionData = {
        exceptionId: this.exception.id,
        decision: decision,
        justification: this.decisionJustification,
        markAsTrusted: this.markAsTrusted,
        notifyAuthor: this.notifyAuthor
      };

      this.$emit('decision-made', decisionData);
    }
  }
};
</script>
```

## 5. Backend e API

### 5.1 Endpoints da API
```python
# /src/api/approval_controllers.py
from flask import Blueprint, request, jsonify
from exceptions.services.approval_service import ApprovalService
from src.auth.decorators import require_ad_group

approval_bp = Blueprint('approval', __name__)
approval_service = ApprovalService()

@approval_bp.route('/api/approval/pending', methods=['GET'])
@require_ad_group()
def get_pending_approvals():
    """
    Obtém lista de exceções pendentes de aprovação
    
    Query Parameters:
    - severity: Filtra por severidade (optional)
    - environment: Filtra por ambiente (optional)
    - limit: Limite de resultados (default: 50)
    - offset: Offset para paginação (default: 0)
    
    Response:
    {
      "exceptions": [...],
      "total": 123,
      "limit": 50,
      "offset": 0
    }
    """
    try:
        filters = {
            'severity': request.args.get('severity'),
            'environment': request.args.get('environment'),
            'limit': int(request.args.get('limit', 50)),
            'offset': int(request.args.get('offset', 0))
        }
        
        result = approval_service.get_pending_approvals(filters)
        return jsonify(result)
    except Exception as e:
        return jsonify({'error': str(e)}), 500

@approval_bp.route('/api/approval/exceptions/<exception_id>', methods=['GET'])
@require_ad_group()
def get_exception_detail(exception_id):
    """
    Obtém detalhes de uma exceção específica
    
    Response:
    {
      "exception": {...},
      "codeContext": "...",
      "authorHistory": {...}
    }
    """
    try:
        detail = approval_service.get_exception_detail(exception_id)
        return jsonify(detail)
    except Exception as e:
        return jsonify({'error': str(e)}), 404

@approval_bp.route('/api/approval/decisions', methods=['POST'])
@require_ad_group()
def record_approval_decision():
    """
    Registra uma decisão de aprovação/rejeição
    
    Request Body:
    {
      "exceptionId": "exc-123",
      "decision": "approve|reject",
      "justification": "Justificativa detalhada",
      "markAsTrusted": true,
      "notifyAuthor": true
    }
    
    Response:
    {
      "status": "success",
      "message": "Decisão registrada com sucesso"
    }
    """
    try:
        data = request.get_json()
        user = request.user  # Obtido do decorator de autenticação
        
        result = approval_service.record_decision(data, user)
        return jsonify(result)
    except Exception as e:
        return jsonify({'error': str(e)}), 400

@approval_bp.route('/api/approval/dashboard/stats', methods=['GET'])
@require_ad_group()
def get_dashboard_stats():
    """
    Obtém estatísticas para o dashboard
    
    Response:
    {
      "pending": 5,
      "approved": 23,
      "rejected": 2,
      "timeout": 1,
      "bySeverity": {
        "critical": 2,
        "high": 3,
        "medium": 0,
        "low": 0
      }
    }
    """
    try:
        stats = approval_service.get_dashboard_statistics()
        return jsonify(stats)
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### 5.2 Serviço de Aprovação
```python
# /exceptions/services/approval_service.py
from exceptions.repositories.exception_repository import ExceptionRepository
from notifications.notification_service import NotificationService
from src.monitoring.approval_metrics import ApprovalMetrics

class ApprovalService:
    def __init__(self):
        self.exception_repo = ExceptionRepository()
        self.notification_service = NotificationService()
        self.metrics = ApprovalMetrics()
    
    def get_pending_approvals(self, filters=None):
        """Obtém exceções pendentes de aprovação"""
        exceptions = self.exception_repo.find_pending_approvals(filters or {})
        
        # Enriquecer com informações adicionais
        enriched_exceptions = []
        for exc in exceptions:
            enriched_exc = self._enrich_exception_data(exc)
            enriched_exceptions.append(enriched_exc)
        
        return {
            'exceptions': enriched_exceptions,
            'total': len(enriched_exceptions)
        }
    
    def get_exception_detail(self, exception_id):
        """Obtém detalhes completos de uma exceção"""
        exception = self.exception_repo.find_by_id(exception_id)
        if not exception:
            raise Exception(f"Exceção {exception_id} não encontrada")
        
        # Obter contexto do código
        code_context = self._get_code_context(
            exception.file_name, 
            exception.line_number
        )
        
        # Obter histórico do autor
        author_history = self._get_author_history(exception.changeset_author)
        
        return {
            'exception': exception.to_dict(),
            'codeContext': code_context,
            'authorHistory': author_history
        }
    
    def record_decision(self, decision_data, approver):
        """Registra uma decisão de aprovação/rejeição"""
        exception_id = decision_data['exceptionId']
        decision = decision_data['decision']
        justification = decision_data['justification']
        
        # Atualizar status da exceção
        self.exception_repo.update_status(
            exception_id, 
            decision,
            approver.username,
            justification
        )
        
        # Atualizar padrões confiáveis se solicitado
        if decision_data.get('markAsTrusted'):
            self._mark_as_trusted(exception_id)
        
        # Notificar interessados
        if decision_data.get('notifyAuthor'):
            self._notify_author_of_decision(exception_id, decision, justification)
        
        # Registrar métricas
        self.metrics.record_approval_decision(
            decision, 
            approver.username,
            decision_data.get('timeToDecision', 0)
        )
        
        return {
            'status': 'success',
            'message': 'Decisão registrada com sucesso'
        }
    
    def get_dashboard_statistics(self):
        """Obtém estatísticas para o dashboard"""
        stats = self.exception_repo.get_approval_statistics()
        return stats
    
    def _enrich_exception_data(self, exception):
        """Enriquece dados da exceção com informações contextuais"""
        # Adicionar informações de severidade formatadas
        exception.display_severity = self._format_severity(exception.calculated_severity)
        
        # Adicionar ícone apropriado
        exception.severity_icon = self._get_severity_icon(exception.calculated_severity)
        
        return exception
    
    def _format_severity(self, severity):
        """Formata o texto da severidade para exibição"""
        severity_map = {
            'low': 'Baixa',
            'medium': 'Média',
            'high': 'Alta',
            'critical': 'Crítica'
        }
        return severity_map.get(severity, severity)
    
    def _get_severity_icon(self, severity):
        """Obtém o ícone apropriado para a severidade"""
        icon_map = {
            'low': '🔵',
            'medium': '🟡',
            'high': '🟠',
            'critical': '🔴'
        }
        return icon_map.get(severity, '⚪')
    
    def _get_code_context(self, file_path, line_number, context_lines=5):
        """Obtém contexto do código ao redor da linha da exceção"""
        try:
            with open(file_path, 'r') as f:
                lines = f.readlines()
            
            start_line = max(0, line_number - context_lines - 1)
            end_line = min(len(lines), line_number + context_lines)
            
            context = []
            for i in range(start_line, end_line):
                line_content = lines[i].rstrip('\n')
                context.append({
                    'lineNumber': i + 1,
                    'content': line_content,
                    'isExceptionLine': (i + 1) == line_number
                })
            
            return context
        except Exception as e:
            return [{'lineNumber': line_number, 'content': f'Erro ao carregar contexto: {str(e)}', 'isExceptionLine': True}]
    
    def _get_author_history(self, author):
        """Obtém histórico de solicitações do autor"""
        return self.exception_repo.get_author_statistics(author)
    
    def _mark_as_trusted(self, exception_id):
        """Marca um padrão como confiável"""
        # Implementação para marcar padrões como confiáveis
        # Isso pode ser usado para auto-aprovar casos similares futuros
        pass
    
    def _notify_author_of_decision(self, exception_id, decision, justification):
        """Notifica o autor sobre a decisão tomada"""
        exception = self.exception_repo.find_by_id(exception_id)
        if exception:
            self.notification_service.send_decision_notification(
                exception.changeset_author,
                decision,
                justification,
                exception
            )
```

## 6. Segurança e Autenticação

### 6.1 Middleware de Autenticação
```python
# /src/auth/decorators.py
from functools import wraps
from flask import request, jsonify
import jwt

def require_ad_group():
    """Decorator para exigir que o usuário pertença ao grupo AD"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Extrair token do header
            auth_header = request.headers.get('Authorization')
            if not auth_header or not auth_header.startswith('Bearer '):
                return jsonify({'error': 'Token de autenticação não fornecido'}), 401
            
            token = auth_header.split(' ')[1]
            
            try:
                # Decodificar token JWT
                payload = jwt.decode(token, 'secret-key', algorithms=['HS256'])
                user_groups = payload.get('groups', [])
                
                # Verificar se usuário pertence ao grupo AD
                if 'AD-GROUP' not in user_groups:
                    return jsonify({'error': 'Acesso negado. Requer permissão de AD-GROUP'}), 403
                
                # Adicionar usuário ao request context
                request.user = {
                    'username': payload.get('sub'),
                    'groups': user_groups,
                    'email': payload.get('email')
                }
                
            except jwt.ExpiredSignatureError:
                return jsonify({'error': 'Token expirado'}), 401
            except jwt.InvalidTokenError:
                return jsonify({'error': 'Token inválido'}), 401
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator
```

### 6.2 Controle de Acesso Granular
```python
# /src/auth/access_control.py
class AccessControl:
    @staticmethod
    def can_approve_exception(user, exception):
        """Verifica se um usuário pode aprovar uma exceção específica"""
        # AD-GROUP pode aprovar tudo
        if 'AD-GROUP' in user.groups:
            return True
        
        # Verificar severidade máxima permitida por grupo
        max_severity = AccessControl._get_max_severity_for_group(user.groups)
        exception_severity_level = AccessControl._get_severity_level(exception.calculated_severity)
        
        return exception_severity_level <= max_severity
    
    @staticmethod
    def _get_max_severity_for_group(groups):
        """Obtém a severidade máxima que um grupo pode aprovar"""
        severity_levels = {
            'AD-GROUP': 4,  # Crítica
            'TEAM-LEADS': 3,  # Alta
            'SENIOR-DEVS': 2,  # Média
            'DEVELOPERS': 1   # Baixa
        }
        
        max_level = 0
        for group in groups:
            level = severity_levels.get(group, 0)
            max_level = max(max_level, level)
        
        return max_level
    
    @staticmethod
    def _get_severity_level(severity):
        """Converte severidade para nível numérico"""
        severity_mapping = {
            'low': 1,
            'medium': 2,
            'high': 3,
            'critical': 4
        }
        return severity_mapping.get(severity, 0)
```

## 7. Notificações e Alertas

### 7.1 Sistema de Notificações
```python
# /notifications/notification_service.py
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from typing import List

class NotificationService:
    def __init__(self, config):
        self.smtp_config = config.get('smtp', {})
        self.template_service = TemplateService()
    
    def send_pending_approval_alert(self, ad_users: List[str], exceptions: List):
        """Envia alerta sobre exceções pendentes de aprovação"""
        subject = f"🚨 {len(exceptions)} Exceções Críticas Aguardando Aprovação"
        
        # Gerar conteúdo do email
        template_data = {
            'exception_count': len(exceptions),
            'exceptions': exceptions,
            'approval_url': '/approval/dashboard'
        }
        
        body = self.template_service.render('pending_approval_alert.html', template_data)
        
        # Enviar para todos os usuários AD
        for user_email in ad_users:
            self._send_email(user_email, subject, body)
    
    def send_decision_notification(self, author_email: str, decision: str, 
                                 justification: str, exception):
        """Notifica o autor sobre a decisão tomada"""
        decision_text = 'aprovada' if decision == 'approve' else 'rejeitada'
        subject = f"📋 Exceção {decision_text} - {exception.changeset_author}:{exception.changeset_id}"
        
        template_data = {
            'decision': decision_text,
            'justification': justification,
            'exception': exception,
            'dashboard_url': '/approval/my-exceptions'
        }
        
        body = self.template_service.render('decision_notification.html', template_data)
        self._send_email(author_email, subject, body)
    
    def send_reminder(self, ad_users: List[str], exception):
        """Envia lembrete sobre exceção pendente"""
        subject = f"⏰ Lembrete: Aprovação Pendente - {exception.changeset_author}:{exception.changeset_id}"
        
        template_data = {
            'exception': exception,
            'due_date': exception.due_date,
            'approval_url': f'/approval/detail/{exception.id}'
        }
        
        body = self.template_service.render('approval_reminder.html', template_data)
        
        for user_email in ad_users:
            self._send_email(user_email, subject, body)
    
    def _send_email(self, recipient: str, subject: str, body: str):
        """Envia email utilizando configuração SMTP"""
        try:
            msg = MIMEMultipart()
            msg['From'] = self.smtp_config.get('sender', 'no-reply@company.com')
            msg['To'] = recipient
            msg['Subject'] = subject
            
            msg.attach(MIMEText(body, 'html'))
            
            server = smtplib.SMTP(
                self.smtp_config.get('server'), 
                self.smtp_config.get('port', 587)
            )
            server.starttls()
            
            if self.smtp_config.get('username'):
                server.login(
                    self.smtp_config.get('username'), 
                    self.smtp_config.get('password')
                )
            
            server.send_message(msg)
            server.quit()
            
        except Exception as e:
            print(f"Erro ao enviar notificação: {e}")
```

## 8. Templates de Email

### 8.1 Alerta de Exceções Pendentes
```html
<!-- /notifications/templates/pending_approval_alert.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Exceções Pendentes de Aprovação</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background-color: #dc3545; color: white; padding: 15px; border-radius: 5px; }
        .exception-list { margin: 20px 0; }
        .exception-item { 
            border: 1px solid #ddd; 
            margin: 10px 0; 
            padding: 15px; 
            border-radius: 5px;
            background-color: #f8f9fa;
        }
        .severity-critical { border-left: 5px solid #dc3545; }
        .severity-high { border-left: 5px solid #fd7e14; }
        .button { 
            background-color: #007bff; 
            color: white; 
            padding: 10px 20px; 
            text-decoration: none; 
            border-radius: 5px;
            display: inline-block;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🚨 {{ exception_count }} Exceções Críticas Aguardando Sua Aprovação</h1>
    </div>
    
    <p>Olá,</p>
    
    <p>Foram detectadas exceções críticas em changesets do Liquibase que requerem sua aprovação imediata:</p>
    
    <div class="exception-list">
        {% for exception in exceptions %}
        <div class="exception-item severity-{{ exception.calculated_severity }}">
            <h3>{{ exception.changeset.author }}:{{ exception.changeset.id }}</h3>
            <p><strong>Regra:</strong> {{ exception.ignored_rule.name }}</p>
            <p><strong>Severidade:</strong> {{ exception.calculated_severity }}</p>
            <p><strong>Justificativa:</strong> {{ exception.justification }}</p>
            <p><strong>Arquivo:</strong> {{ exception.file_name }}</p>
        </div>
        {% endfor %}
    </div>
    
    <p>
        <a href="{{ approval_url }}" class="button">Acessar Painel de Aprovações</a>
    </p>
    
    <p>Por favor, revise estas exceções o quanto antes para não impactar o pipeline de CI/CD.</p>
    
    <p>Atenciosamente,<br>Equipe de Governança de Dados</p>
</body>
</html>
```

## 9. Mobile Responsiveness

### 9.1 Media Queries para Responsividade
```css
/* /src/web/assets/css/approval-responsive.css */
@media (max-width: 768px) {
    .dashboard-header {
        flex-direction: column;
        align-items: stretch;
    }
    
    .header-controls {
        margin-top: 10px;
        justify-content: space-between;
    }
    
    .summary-cards {
        grid-template-columns: 1fr 1fr;
        gap: 10px;
    }
    
    .exception-item {
        padding: 10px;
    }
    
    .exception-item h3 {
        font-size: 16px;
    }
    
    .decision-form textarea {
        height: 100px;
    }
    
    .decision-buttons {
        flex-direction: column;
        gap: 10px;
    }
    
    .decision-buttons button {
        width: 100%;
    }
}

@media (max-width: 480px) {
    .summary-cards {
        grid-template-columns: 1fr;
    }
    
    .exception-details .details-grid {
        grid-template-columns: 1fr;
    }
    
    .changeset-info .info-grid {
        grid-template-columns: 1fr;
    }
    
    .modal-overlay {
        padding: 10px;
    }
    
    .exception-detail-modal {
        max-height: 90vh;
    }
}
```

## 10. Considerações de UX

### 10.1 Princípios de Design
1. **Clareza Visual**: Uso de cores e ícones para indicar severidade
2. **Hierarquia de Informação**: Informações mais importantes destacadas
3. **Redução de Ruído**: Apenas informações relevantes exibidas
4. **Feedback Imediato**: Confirmações visuais para ações
5. **Acessibilidade**: Contraste adequado e navegação por teclado

### 10.2 Estados de UI
1. **Loading State**: Spinner e mensagem durante carregamento
2. **Empty State**: Mensagem quando não há exceções pendentes
3. **Error State**: Tratamento e exibição de erros
4. **Success State**: Confirmação de ações bem-sucedidas

Esta interface de aprovação para o grupo AD fornece uma experiência completa e eficiente para gestão de exceções críticas `linter-ignore-rule`, com foco em segurança, usabilidade e integração completa com o sistema de governança existente.