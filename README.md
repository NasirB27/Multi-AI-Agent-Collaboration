"""
AutoGen Basic Agent Implementation

This code demonstrates how to create a basic agent using the AutoGen AI Framework.
It sets up two agents (assistant and user proxy) that can collaborate to solve tasks.
"""

import autogen
from typing import Dict, Optional, Union
import os

# Configuration for Azure OpenAI
config_list = [
    {
        "model": "gpt-4",
        "api_key": os.environ.get("AZURE_OPENAI_API_KEY"),
        "api_base": os.environ.get("AZURE_OPENAI_API_BASE"),
        "api_type": "azure",
        "api_version": "2023-07-01-preview"
    }
]

# Create agent configurations with proper error handling
def create_agent_config(model_config: Dict) -> Dict:
    """Create a configuration for the agent with proper error handling."""
    if not model_config.get("api_key"):
        raise ValueError("API key is required but not provided")
    
    # Set reasonable defaults for missing configuration
    return {
        "temperature": 0.7,
        "max_tokens": 1000,
        "timeout": 600,
        "cache_seed": 42,  # For reproducibility
        **model_config
    }

# Initialize the assistant agent
try:
    assistant = autogen.AssistantAgent(
        name="assistant",
        system_message="You are a helpful AI assistant. Solve tasks step by step.",
        llm_config={"config_list": [create_agent_config(config) for config in config_list]}
    )
except Exception as e:
    print(f"Error initializing assistant agent: {e}")
    exit(1)

# Initialize the user proxy agent
try:
    user_proxy = autogen.UserProxyAgent(
        name="user_proxy",
        human_input_mode="TERMINATE",  # Only ask for human input when the assistant requests termination
        max_consecutive_auto_reply=10,  # Limit auto-replies to prevent infinite loops
        system_message="You are a user with a task. Work with the assistant to solve it."
    )
except Exception as e:
    print(f"Error initializing user proxy agent: {e}")
    exit(1)

def run_agent_conversation(task: str) -> None:
    """Run a conversation between agents to solve the given task."""
    try:
        # Initiate the conversation
        user_proxy.initiate_chat(
            assistant,
            message=task
        )
        print("Task completed successfully")
    except Exception as e:
        print(f"Error during agent conversation: {e}")

# Example usage
if __name__ == "__main__":
    # Define a sample task
    task = "Explain how to implement a binary search algorithm and provide Python code."
    
    print(f"Starting agent conversation for task: {task}")
    run_agent_conversation(task)
