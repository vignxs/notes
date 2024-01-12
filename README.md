from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnableParallel
from typing import Any, Dict, Optional
from langchain_core.load.dump import dumpd, dumps
from langchain_core.runnables.config import (
    RunnableConfig,
    ensure_config,
    get_executor_for_config,
    patch_config,
)


necessary_tool_names = ['sql_db_query', 'sql_db_schema']
    agent_executor.tools = [tool for tool in agent_executor.tools if tool.name in necessary_tool_names]

    
class CustomRunnableParallel(RunnableParallel):
    def invoke(self, inputs: Dict[str, Any], config: Optional[RunnableConfig] = None) -> Dict[str, Any]:
        from langchain_core.callbacks.manager import CallbackManager

        # Setup callbacks
        config = ensure_config(config)
        callback_manager = CallbackManager.configure(
            inheritable_callbacks=config.get("callbacks"),
            local_callbacks=None,
            verbose=False,
            inheritable_tags=config.get("tags"),
            local_tags=None,
            inheritable_metadata=config.get("metadata"),
            local_metadata=None,
        )
        # Start the root run
        run_manager = callback_manager.on_chain_start(
            dumpd(self), inputs, name=config.get("run_name")
        )

        try:
            steps = dict(self.steps)
            with get_executor_for_config(config) as executor:
                futures = []
                for key, step in steps.items():
                    step_input = inputs.get(key, {}).get('topic')  # Extract the 'topic' from the specific input
                    future = executor.submit(
                        step.invoke,
                        {'topic': step_input},  # Pass 'topic' as expected by the step
                        patch_config(
                            config,
                            callbacks=run_manager.get_child(f"map:key:{key}"),
                        ),
                    )
                    futures.append((key, future))
                output = {key: future.result() for key, future in futures}
        except BaseException as e:
            run_manager.on_chain_error(e)
            raise
        else:
            run_manager.on_chain_end(output)
            return output



custom_map_chain = CustomRunnableParallel(joke=joke_chain, poem=poem_chain)
result = custom_map_chain.invoke({
    "joke": {"topic": "bear"},
    "poem": {"topic": "sun"}
})

result

Accomplishments in 2023:
I led the successful implementation of natural language SQL interactions using Langchain and OpenAI, significantly enhancing data accessibility for non-technical users. My successful demo to the CEO and stakeholders underscored the benefits in decision-making and operational efficiency.
I overhauled our ETL pipeline to generate comprehensive historical reports from 2015 to present, facilitating deeper composition analysis. My initiative not only improved reporting accuracy but also increased team productivity through Knowledge Transfer (KT) sessions.
Took charge of the Ripa Chatbot project, ensuring alignment with sprint requirements and expectations. Regularly showcased progress to stakeholders, maintaining transparency and meeting project milestones.
Improved reports in the UI by introducing exception handling and error comments within the ETL pipeline. Elevated the overall robustness of the system, providing clearer insights into data processing challenges.




