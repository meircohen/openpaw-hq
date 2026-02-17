# OpenPaw API Contracts and Interface Definitions

**Version:** 1.0  
**Date:** February 17, 2026  
**Target:** OpenPaw Mac Application v1.0  

## Overview

This document defines every internal API and interface between OpenPaw components. All message formats use JSON Schema for precise specification. Engineers can implement both sides independently using these contracts.

## WebSocket Communication Protocol

### Connection and Authentication

#### Connection URL
```
ws://127.0.0.1:3002/ws
```

#### Authentication Message

**Type:** `authenticate`  
**Direction:** Swift UI → OpenClaw Gateway  
**Timing:** Immediately after connection established

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique message identifier"
    },
    "type": {
      "type": "string",
      "enum": ["authenticate"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "clientType": {
          "type": "string",
          "enum": ["openpaw"]
        },
        "version": {
          "type": "string",
          "pattern": "^\\d+\\.\\d+\\.\\d+$"
        },
        "sessionId": {
          "type": "string",
          "format": "uuid"
        },
        "capabilities": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["task_submission", "approval_flow", "status_updates", "memory_query"]
          }
        }
      },
      "required": ["clientType", "version", "sessionId", "capabilities"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Authentication Response

**Type:** `authenticate_response`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["authenticate_response"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "status": {
          "type": "string",
          "enum": ["success", "error"]
        },
        "gatewayVersion": {
          "type": "string"
        },
        "agentStatus": {
          "type": "string",
          "enum": ["ready", "initializing", "error"]
        },
        "supportedCapabilities": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "error": {
          "type": "object",
          "properties": {
            "code": {"type": "string"},
            "message": {"type": "string"}
          }
        }
      },
      "required": ["status", "gatewayVersion", "agentStatus"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Task Submission and Execution

#### Task Submission

**Type:** `submit_task`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["submit_task"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "taskId": {
          "type": "string",
          "format": "uuid",
          "description": "Client-generated unique task identifier"
        },
        "userMessage": {
          "type": "string",
          "description": "User's request/message",
          "maxLength": 10000
        },
        "context": {
          "type": "object",
          "properties": {
            "channel": {
              "type": "string",
              "enum": ["ui", "voice", "text"]
            },
            "attachments": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "enum": ["file", "image", "url"]
                  },
                  "path": {"type": "string"},
                  "mimeType": {"type": "string"},
                  "size": {"type": "integer"}
                },
                "required": ["type", "path"]
              }
            },
            "priority": {
              "type": "string",
              "enum": ["low", "normal", "high", "urgent"],
              "default": "normal"
            },
            "requiresApproval": {
              "type": "boolean",
              "default": false
            }
          }
        },
        "settings": {
          "type": "object",
          "properties": {
            "model": {
              "type": "string",
              "enum": ["sonnet-4", "opus-4.6", "gemini-2.5-pro", "haiku-4.5"]
            },
            "maxTokens": {
              "type": "integer",
              "minimum": 1000,
              "maximum": 100000
            },
            "temperature": {
              "type": "number",
              "minimum": 0.0,
              "maximum": 1.0
            }
          }
        }
      },
      "required": ["taskId", "userMessage"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Task Acknowledgment

**Type:** `task_acknowledged`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["task_acknowledged"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "taskId": {
          "type": "string",
          "format": "uuid"
        },
        "estimatedDuration": {
          "type": "integer",
          "description": "Estimated completion time in seconds"
        },
        "assignedAgent": {
          "type": "string",
          "description": "Agent handling this task"
        }
      },
      "required": ["taskId"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Task Status and Progress Updates

#### Task Status Update

**Type:** `task_status`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["task_status"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "taskId": {
          "type": "string",
          "format": "uuid"
        },
        "status": {
          "type": "string",
          "enum": ["queued", "running", "paused", "completed", "failed", "cancelled"]
        },
        "progress": {
          "type": "object",
          "properties": {
            "percentage": {
              "type": "integer",
              "minimum": 0,
              "maximum": 100
            },
            "currentStep": {
              "type": "string"
            },
            "totalSteps": {
              "type": "integer"
            },
            "stepProgress": {
              "type": "integer",
              "minimum": 0,
              "maximum": 100
            }
          }
        },
        "result": {
          "type": "object",
          "properties": {
            "success": {
              "type": "boolean"
            },
            "message": {
              "type": "string"
            },
            "data": {
              "type": "object"
            },
            "artifacts": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "enum": ["file", "url", "text", "image"]
                  },
                  "path": {"type": "string"},
                  "title": {"type": "string"},
                  "description": {"type": "string"}
                }
              }
            }
          }
        },
        "metrics": {
          "type": "object",
          "properties": {
            "tokensUsed": {"type": "integer"},
            "executionTime": {"type": "integer"},
            "toolCalls": {"type": "integer"},
            "cost": {"type": "number"}
          }
        }
      },
      "required": ["taskId", "status"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Task Progress Update

**Type:** `task_progress`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["task_progress"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "taskId": {
          "type": "string",
          "format": "uuid"
        },
        "step": {
          "type": "string",
          "description": "Current step description"
        },
        "details": {
          "type": "string",
          "description": "Detailed progress information"
        },
        "toolCall": {
          "type": "object",
          "properties": {
            "tool": {"type": "string"},
            "action": {"type": "string"},
            "parameters": {"type": "object"}
          }
        },
        "thinking": {
          "type": "string",
          "description": "Agent reasoning (if enabled)"
        }
      },
      "required": ["taskId", "step"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Action Approval Flow

#### Approval Request

**Type:** `approval_request`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["approval_request"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "taskId": {
          "type": "string",
          "format": "uuid"
        },
        "approvalId": {
          "type": "string",
          "format": "uuid"
        },
        "action": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["file_write", "file_delete", "email_send", "api_call", "shell_command", "financial_transaction"]
            },
            "description": {
              "type": "string",
              "description": "Human-readable action description"
            },
            "details": {
              "type": "object",
              "properties": {
                "tool": {"type": "string"},
                "parameters": {"type": "object"},
                "riskLevel": {
                  "type": "string",
                  "enum": ["low", "medium", "high", "critical"]
                },
                "reversible": {"type": "boolean"},
                "consequences": {
                  "type": "array",
                  "items": {"type": "string"}
                }
              }
            }
          },
          "required": ["type", "description", "details"]
        },
        "context": {
          "type": "object",
          "properties": {
            "reasoning": {"type": "string"},
            "alternatives": {
              "type": "array",
              "items": {"type": "string"}
            },
            "relatedFiles": {
              "type": "array",
              "items": {"type": "string"}
            }
          }
        },
        "timeout": {
          "type": "integer",
          "description": "Approval timeout in seconds",
          "default": 300
        }
      },
      "required": ["taskId", "approvalId", "action"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Approval Response

**Type:** `approval_response`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["approval_response"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "approvalId": {
          "type": "string",
          "format": "uuid"
        },
        "decision": {
          "type": "string",
          "enum": ["approve", "deny", "modify"]
        },
        "modifications": {
          "type": "object",
          "description": "Modified parameters if decision is 'modify'"
        },
        "reason": {
          "type": "string",
          "description": "User's reason for the decision"
        },
        "rememberChoice": {
          "type": "boolean",
          "description": "Whether to remember this decision for similar future actions"
        }
      },
      "required": ["approvalId", "decision"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Settings Interface

#### Settings Read Request

**Type:** `get_settings`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["get_settings"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "categories": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["agent", "integrations", "security", "workspace", "ui", "all"]
          },
          "default": ["all"]
        }
      }
    }
  },
  "required": ["id", "type", "timestamp"]
}
```

#### Settings Response

**Type:** `settings_response`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["settings_response"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "settings": {
          "type": "object",
          "properties": {
            "agent": {
              "type": "object",
              "properties": {
                "name": {"type": "string"},
                "model": {"type": "string"},
                "personality": {"type": "string"},
                "temperature": {"type": "number"},
                "maxTokens": {"type": "integer"},
                "thinkingMode": {
                  "type": "string",
                  "enum": ["off", "low", "on", "stream"]
                }
              }
            },
            "integrations": {
              "type": "object",
              "properties": {
                "oauth": {
                  "type": "object",
                  "additionalProperties": {
                    "type": "object",
                    "properties": {
                      "enabled": {"type": "boolean"},
                      "scopes": {
                        "type": "array",
                        "items": {"type": "string"}
                      },
                      "lastRefresh": {"type": "string"}
                    }
                  }
                },
                "apis": {
                  "type": "object",
                  "additionalProperties": {
                    "type": "object",
                    "properties": {
                      "enabled": {"type": "boolean"},
                      "endpoint": {"type": "string"},
                      "rateLimit": {"type": "integer"}
                    }
                  }
                }
              }
            },
            "security": {
              "type": "object",
              "properties": {
                "requireApproval": {
                  "type": "array",
                  "items": {
                    "type": "string",
                    "enum": ["file_operations", "external_communications", "financial", "system_changes"]
                  }
                },
                "allowedHosts": {
                  "type": "array",
                  "items": {"type": "string"}
                },
                "toolPolicy": {
                  "type": "string",
                  "enum": ["restricted", "standard", "permissive"]
                }
              }
            },
            "workspace": {
              "type": "object",
              "properties": {
                "path": {"type": "string"},
                "autoBackup": {"type": "boolean"},
                "maxMemoryDays": {"type": "integer"},
                "logLevel": {
                  "type": "string",
                  "enum": ["debug", "info", "warn", "error"]
                }
              }
            }
          }
        }
      },
      "required": ["settings"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Settings Update

**Type:** `update_settings`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["update_settings"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "updates": {
          "type": "object",
          "description": "Partial settings object with only changed values"
        },
        "restartRequired": {
          "type": "boolean",
          "description": "Whether gateway restart is required for changes"
        }
      },
      "required": ["updates"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Memory and History Interface

#### Memory Query

**Type:** `query_memory`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["query_memory"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "query": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string",
              "enum": ["search", "recent", "date_range", "by_task"]
            },
            "parameters": {
              "type": "object",
              "properties": {
                "searchTerm": {"type": "string"},
                "dateFrom": {"type": "string", "format": "date"},
                "dateTo": {"type": "string", "format": "date"},
                "taskId": {"type": "string"},
                "limit": {"type": "integer", "default": 50},
                "offset": {"type": "integer", "default": 0}
              }
            }
          },
          "required": ["type"]
        }
      },
      "required": ["query"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

#### Memory Response

**Type:** `memory_response`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["memory_response"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "results": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "id": {"type": "string"},
              "timestamp": {"type": "string", "format": "date-time"},
              "type": {
                "type": "string",
                "enum": ["task", "conversation", "decision", "learning"]
              },
              "content": {"type": "string"},
              "metadata": {
                "type": "object",
                "properties": {
                  "taskId": {"type": "string"},
                  "importance": {
                    "type": "string",
                    "enum": ["low", "medium", "high"]
                  },
                  "tags": {
                    "type": "array",
                    "items": {"type": "string"}
                  },
                  "relatedEntities": {
                    "type": "array",
                    "items": {"type": "string"}
                  }
                }
              },
              "relevanceScore": {
                "type": "number",
                "minimum": 0,
                "maximum": 1
              }
            },
            "required": ["id", "timestamp", "type", "content"]
          }
        },
        "totalResults": {"type": "integer"},
        "hasMore": {"type": "boolean"}
      },
      "required": ["results", "totalResults", "hasMore"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Health Check and Status

#### Health Check Request

**Type:** `health_check`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["health_check"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "includeMetrics": {
          "type": "boolean",
          "default": false
        }
      }
    }
  },
  "required": ["id", "type", "timestamp"]
}
```

#### Health Status Response

**Type:** `health_status`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["health_status"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "status": {
          "type": "string",
          "enum": ["healthy", "degraded", "unhealthy"]
        },
        "uptime": {
          "type": "integer",
          "description": "Uptime in seconds"
        },
        "version": {"type": "string"},
        "components": {
          "type": "object",
          "properties": {
            "agent": {
              "type": "string",
              "enum": ["ready", "busy", "error"]
            },
            "workspace": {
              "type": "string",
              "enum": ["accessible", "error"]
            },
            "skills": {
              "type": "string",
              "enum": ["loaded", "loading", "error"]
            },
            "integrations": {
              "type": "string",
              "enum": ["connected", "partial", "disconnected"]
            }
          }
        },
        "metrics": {
          "type": "object",
          "properties": {
            "memoryUsage": {
              "type": "object",
              "properties": {
                "used": {"type": "integer"},
                "total": {"type": "integer"},
                "percentage": {"type": "number"}
              }
            },
            "cpuUsage": {
              "type": "number",
              "minimum": 0,
              "maximum": 100
            },
            "activeConnections": {"type": "integer"},
            "tasksInQueue": {"type": "integer"},
            "totalTasksProcessed": {"type": "integer"}
          }
        }
      },
      "required": ["status", "uptime", "version", "components"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Error Reporting

#### Error Report

**Type:** `error_report`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["error_report"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "error": {
          "type": "object",
          "properties": {
            "code": {
              "type": "string",
              "enum": ["TASK_FAILED", "CONFIG_ERROR", "SKILL_ERROR", "INTEGRATION_ERROR", "SYSTEM_ERROR", "TIMEOUT", "AUTH_ERROR"]
            },
            "severity": {
              "type": "string",
              "enum": ["low", "medium", "high", "critical"]
            },
            "message": {
              "type": "string",
              "description": "Human-readable error message"
            },
            "details": {
              "type": "object",
              "properties": {
                "stackTrace": {"type": "string"},
                "component": {"type": "string"},
                "taskId": {"type": "string"},
                "tool": {"type": "string"}
              }
            },
            "context": {
              "type": "object",
              "properties": {
                "userAction": {"type": "string"},
                "systemState": {"type": "object"},
                "environment": {"type": "object"}
              }
            }
          },
          "required": ["code", "severity", "message"]
        },
        "recovery": {
          "type": "object",
          "properties": {
            "suggested": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "action": {"type": "string"},
                  "description": {"type": "string"},
                  "automatic": {"type": "boolean"}
                }
              }
            },
            "attempted": {
              "type": "array",
              "items": {"type": "string"}
            }
          }
        }
      },
      "required": ["error"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

## Onboarding Flow Data Model

### Onboarding State

**File:** `~/Library/Application Support/OpenPaw/onboarding-state.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "version": {"type": "string"},
    "currentStep": {
      "type": "string",
      "enum": ["welcome", "user_profile", "agent_config", "integrations", "security", "skills", "complete"]
    },
    "completedSteps": {
      "type": "array",
      "items": {"type": "string"}
    },
    "userProfile": {
      "type": "object",
      "properties": {
        "name": {"type": "string"},
        "role": {"type": "string"},
        "company": {"type": "string"},
        "timezone": {"type": "string"},
        "location": {"type": "string"},
        "workHours": {
          "type": "object",
          "properties": {
            "start": {"type": "string", "pattern": "^\\d{2}:\\d{2}$"},
            "end": {"type": "string", "pattern": "^\\d{2}:\\d{2}$"},
            "workdays": {
              "type": "array",
              "items": {
                "type": "string",
                "enum": ["monday", "tuesday", "wednesday", "thursday", "friday", "saturday", "sunday"]
              }
            }
          }
        },
        "communicationPreferences": {
          "type": "object",
          "properties": {
            "style": {
              "type": "string",
              "enum": ["formal", "casual", "direct", "detailed"]
            },
            "frequency": {
              "type": "string",
              "enum": ["minimal", "moderate", "frequent"]
            },
            "channels": {
              "type": "array",
              "items": {
                "type": "string",
                "enum": ["ui", "notifications", "email"]
              }
            }
          }
        },
        "priorities": {
          "type": "array",
          "items": {"type": "string"}
        },
        "goals": {
          "type": "array",
          "items": {"type": "string"}
        },
        "backgroundContext": {"type": "string"}
      },
      "required": ["name", "timezone"]
    },
    "agentConfig": {
      "type": "object",
      "properties": {
        "name": {"type": "string"},
        "personality": {
          "type": "object",
          "properties": {
            "traits": {
              "type": "array",
              "items": {"type": "string"}
            },
            "communicationStyle": {"type": "string"},
            "expertise": {
              "type": "array",
              "items": {"type": "string"}
            },
            "restrictions": {
              "type": "array",
              "items": {"type": "string"}
            }
          }
        },
        "model": {
          "type": "string",
          "enum": ["sonnet-4", "opus-4.6", "gemini-2.5-pro", "haiku-4.5"]
        },
        "capabilities": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["web_search", "email", "calendar", "file_operations", "code_execution", "financial_tracking"]
          }
        }
      },
      "required": ["name", "model"]
    },
    "integrations": {
      "type": "object",
      "properties": {
        "oauth": {
          "type": "object",
          "additionalProperties": {
            "type": "object",
            "properties": {
              "enabled": {"type": "boolean"},
              "scopes": {
                "type": "array",
                "items": {"type": "string"}
              },
              "tokens": {
                "type": "object",
                "properties": {
                  "access_token": {"type": "string"},
                  "refresh_token": {"type": "string"},
                  "expires_at": {"type": "integer"}
                }
              }
            }
          }
        },
        "apis": {
          "type": "object",
          "additionalProperties": {
            "type": "object",
            "properties": {
              "enabled": {"type": "boolean"},
              "apiKey": {"type": "string"},
              "endpoint": {"type": "string"},
              "settings": {"type": "object"}
            }
          }
        }
      }
    },
    "security": {
      "type": "object",
      "properties": {
        "requireApproval": {
          "type": "array",
          "items": {
            "type": "string",
            "enum": ["file_operations", "external_communications", "financial", "system_changes", "data_access"]
          }
        },
        "allowedHosts": {
          "type": "array",
          "items": {"type": "string"}
        },
        "toolPolicy": {
          "type": "string",
          "enum": ["restricted", "standard", "permissive"]
        },
        "dataRetention": {
          "type": "object",
          "properties": {
            "memoryDays": {"type": "integer"},
            "logDays": {"type": "integer"},
            "backupEnabled": {"type": "boolean"}
          }
        }
      }
    },
    "skills": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": {"type": "string"},
          "name": {"type": "string"},
          "category": {"type": "string"},
          "enabled": {"type": "boolean"},
          "config": {"type": "object"}
        }
      }
    }
  },
  "required": ["version", "currentStep", "completedSteps"]
}
```

### Onboarding Completion Event

When onboarding is complete, OpenPaw generates all required files and configurations:

1. **Generate openclaw.json** from onboarding state
2. **Generate SOUL.md** from agent personality configuration  
3. **Generate USER.md** from user profile
4. **Generate AGENTS.md** with default agent instructions
5. **Install selected skills**
6. **Configure OAuth integrations**
7. **Set security policies**

## Task Management Data Models

### Task Queue Entry

**File:** `~/Library/Application Support/OpenPaw/workspace/config/task-queue.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "taskId": {
        "type": "string",
        "format": "uuid"
      },
      "status": {
        "type": "string",
        "enum": ["queued", "running", "paused", "completed", "failed", "cancelled"]
      },
      "priority": {
        "type": "string",
        "enum": ["low", "normal", "high", "urgent"]
      },
      "createdAt": {
        "type": "string",
        "format": "date-time"
      },
      "startedAt": {
        "type": "string",
        "format": "date-time"
      },
      "completedAt": {
        "type": "string",
        "format": "date-time"
      },
      "userMessage": {"type": "string"},
      "assignedAgent": {"type": "string"},
      "estimatedDuration": {"type": "integer"},
      "actualDuration": {"type": "integer"},
      "retryCount": {"type": "integer", "default": 0},
      "maxRetries": {"type": "integer", "default": 3},
      "context": {
        "type": "object",
        "properties": {
          "channel": {"type": "string"},
          "attachments": {"type": "array"},
          "requiresApproval": {"type": "boolean"}
        }
      },
      "result": {
        "type": "object",
        "properties": {
          "success": {"type": "boolean"},
          "message": {"type": "string"},
          "data": {"type": "object"},
          "error": {
            "type": "object",
            "properties": {
              "code": {"type": "string"},
              "message": {"type": "string"},
              "stackTrace": {"type": "string"}
            }
          }
        }
      },
      "metrics": {
        "type": "object",
        "properties": {
          "tokensUsed": {"type": "integer"},
          "toolCalls": {"type": "integer"},
          "cost": {"type": "number"}
        }
      }
    },
    "required": ["taskId", "status", "priority", "createdAt", "userMessage"]
  }
}
```

## Configuration Management

### User Settings Schema

**File:** `~/Library/Application Support/OpenPaw/user-settings.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "app": {
      "type": "object",
      "properties": {
        "theme": {
          "type": "string",
          "enum": ["light", "dark", "auto"]
        },
        "language": {"type": "string"},
        "startMinimized": {"type": "boolean"},
        "showNotifications": {"type": "boolean"},
        "soundEnabled": {"type": "boolean"}
      }
    },
    "agent": {
      "type": "object",
      "properties": {
        "autoStart": {"type": "boolean"},
        "heartbeatInterval": {"type": "integer"},
        "maxConcurrentTasks": {"type": "integer"},
        "defaultModel": {"type": "string"},
        "thinkingMode": {
          "type": "string",
          "enum": ["off", "low", "on", "stream"]
        }
      }
    },
    "ui": {
      "type": "object",
      "properties": {
        "showThinking": {"type": "boolean"},
        "showProgress": {"type": "boolean"},
        "chatFontSize": {"type": "integer"},
        "codeHighlighting": {"type": "boolean"},
        "markdownRendering": {"type": "boolean"}
      }
    },
    "security": {
      "type": "object",
      "properties": {
        "requireApprovalTimeout": {"type": "integer"},
        "rememberApprovalChoices": {"type": "boolean"},
        "allowDataCollection": {"type": "boolean"},
        "encryptWorkspace": {"type": "boolean"}
      }
    }
  }
}
```

## Skill Management API

### Skill Installation Request

**Type:** `install_skill`  
**Direction:** Swift UI → OpenClaw Gateway

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["install_skill"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "skillId": {"type": "string"},
        "version": {"type": "string"},
        "source": {
          "type": "string",
          "enum": ["registry", "local", "github"]
        },
        "config": {
          "type": "object",
          "description": "Initial configuration for the skill"
        }
      },
      "required": ["skillId", "source"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

### Skill Status Response

**Type:** `skill_status`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["skill_status"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "skillId": {"type": "string"},
        "status": {
          "type": "string",
          "enum": ["downloading", "installing", "configuring", "installed", "failed", "disabled"]
        },
        "progress": {
          "type": "integer",
          "minimum": 0,
          "maximum": 100
        },
        "error": {
          "type": "object",
          "properties": {
            "code": {"type": "string"},
            "message": {"type": "string"}
          }
        },
        "metadata": {
          "type": "object",
          "properties": {
            "name": {"type": "string"},
            "version": {"type": "string"},
            "description": {"type": "string"},
            "author": {"type": "string"},
            "capabilities": {
              "type": "array",
              "items": {"type": "string"}
            }
          }
        }
      },
      "required": ["skillId", "status"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

## Real-time Event Streaming

### Event Stream Message

**Type:** `event_stream`  
**Direction:** OpenClaw Gateway → Swift UI

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["event_stream"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "payload": {
      "type": "object",
      "properties": {
        "eventType": {
          "type": "string",
          "enum": ["system", "task", "agent", "user", "integration"]
        },
        "event": {
          "type": "object",
          "properties": {
            "name": {"type": "string"},
            "data": {"type": "object"},
            "metadata": {
              "type": "object",
              "properties": {
                "taskId": {"type": "string"},
                "component": {"type": "string"},
                "severity": {"type": "string"}
              }
            }
          },
          "required": ["name"]
        }
      },
      "required": ["eventType", "event"]
    }
  },
  "required": ["id", "type", "timestamp", "payload"]
}
```

## Connection Management

### Heartbeat Messages

**Type:** `ping` / `pong`  
**Direction:** Bidirectional

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["ping", "pong"]
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    }
  },
  "required": ["id", "type", "timestamp"]
}
```

### Connection State Events

#### Connection Lost

Triggered when WebSocket connection is lost. UI should show offline state and attempt reconnection.

#### Reconnection Success

Triggered when WebSocket connection is re-established. UI should sync state and resume normal operation.

## Implementation Guidelines

### Message Validation

All implementations MUST validate incoming messages against their JSON schemas before processing. Invalid messages should be rejected with appropriate error responses.

### Error Handling

- Use appropriate HTTP-style status codes in error responses
- Include human-readable error messages
- Provide specific error codes for programmatic handling
- Include context information when available

### Performance Requirements

- Message acknowledgment within 100ms
- Task status updates every 5 seconds for long-running tasks
- WebSocket ping/pong every 30 seconds
- Maximum message size: 10MB
- Connection timeout: 60 seconds

### Security Considerations

- All communication over localhost only
- No external network access for WebSocket communication
- Validate all input parameters
- Sanitize user content before processing
- Encrypt sensitive data in configurations

### Versioning Strategy

- All messages include schema version in `$schema` field
- Backward compatibility maintained for at least 2 major versions
- New message types can be added without breaking changes
- Required field changes constitute breaking changes

---

**Document Status:** Draft v1.0  
**Next Review:** March 1, 2026  
**Implementation Target:** Q2 2026

This specification provides the complete API contract between OpenPaw components. Any changes to these interfaces must be reflected in both the Swift UI and OpenClaw Gateway implementations simultaneously.