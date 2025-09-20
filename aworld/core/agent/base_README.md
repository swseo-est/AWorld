# Core Agent Module (`base.py`)

This script defines the foundational components for all agents within the AWorld framework. It provides the abstract base class `BaseAgent` and the factory `AgentManager` for creating and managing agents.

## Key Components

### `BaseAgent` (Abstract Base Class)

`BaseAgent` is the cornerstone of the agent system. Any custom agent created within the framework must inherit from this class.

#### Core Responsibilities:
-   **Configuration**: The `__init__` method handles a wide range of configuration options, including the agent's `name`, `description`, LLM settings (`conf`), and the tools it is allowed to use (`tool_names`, `mcp_servers`).
-   **Identity**: Each agent has a unique `id` and a `name`. The `desc` is used when the agent itself is exposed as a tool to other agents.
-   **Tool Management**: It automatically initializes a `Sandbox` instance if the agent is equipped with tools, ensuring that tool execution is handled safely.
-   **State Management**: It defines an agent's status using the `AgentStatus` class.
-   **Execution Lifecycle**: The `run()` and `async_run()` methods orchestrate the process of an agent taking a single turn. This involves pre-run hooks, executing the core policy, and post-run hooks.

#### The `policy()` Method
The most critical part of `BaseAgent` is the abstract `policy()` method (and its async counterpart `async_policy()`).

```python
@abc.abstractmethod
def policy(self, observation: INPUT, info: Dict[str, Any] = None, **kwargs) -> OUTPUT:
    # ...
```

This method must be implemented by every concrete agent class. It contains the core decision-making logic of the agent. It takes the current `observation` (the input from the environment or previous step) and returns an `AgentResult` object, which specifies the action(s) the agent intends to take.

---

### `AgentResult` (Pydantic Model)

This class defines the standardized structure for the output of an agent's `policy` method. It ensures that the framework can reliably understand the agent's intentions.

-   `actions`: A list of `ActionModel` objects, where each object represents a tool call or a handoff to another agent.
-   `is_call_tool`: A boolean indicating whether the agent is trying to use a tool.

---

### `AgentManager` and `AgentFactory`

`AgentManager` is a factory class that provides a centralized system for registering, configuring, and instantiating agents.

-   **Registration**: Developers can register their custom agent classes with the factory, associating them with a name, a description, and a default configuration file.
-   **Instantiation**: Agents can then be created by name using the factory, which automatically loads their default configuration and passes in any overrides.
-   **Singleton**: The framework uses a global singleton instance of `AgentManager` called `AgentFactory`, making it easy to access from anywhere in the codebase.

This factory pattern is key to the framework's extensibility, allowing new agents to be added and used without modifying the core framework code.
