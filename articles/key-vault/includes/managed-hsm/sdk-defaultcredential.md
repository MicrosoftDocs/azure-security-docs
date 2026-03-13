`DefaultAzureCredential` automatically selects the appropriate credential based on your environment:

| Environment | Credential used |
|-------------|-----------------|
| Azure VMs, App Service, Functions | System-assigned or user-assigned managed identity |
| Azure Kubernetes Service | Workload identity |
| Local development | Azure CLI, Visual Studio, or VS Code credentials |
| CI/CD pipelines | Workload identity federation or service principal |

The credential checks these sources in order:
1. Environment variables
2. Workload identity
3. Managed identity
4. Azure CLI
5. Azure PowerShell
6. Visual Studio / VS Code credentials

For production workloads in Azure, managed identities are strongly recommended because they eliminate credential management entirely.
