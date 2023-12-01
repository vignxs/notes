Sure, I can help you create an inherited class from RunnableParallel and modify the invoke method to accommodate different inputs for each chain. The new class will override the invoke method to handle a dictionary of inputs, where each key corresponds to a specific chain and each value is the input for that chain.

Here's an example of how you can achieve this:

python
Copy code
from langchain.schema.runnable import RunnableParallel
from typing import Any, Dict, Optional

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
                futures = [
                    executor.submit(
                        step.invoke,
                        inputs.get(key, None),  # Pass specific input to each step
                        patch_config(
                            config,
                            callbacks=run_manager.get_child(f"map:key:{key}"),
                        ),
                    )
                    for key, step in steps.items()
                ]
                output = {key: future.result() for key, future in zip(steps, futures)}
        except BaseException as e:
            run_manager.on_chain_error(e)
            raise
        else:
            run_manager.on_chain_end(output)
            return output

# Usage example
custom_map_chain = CustomRunnableParallel(joke=joke_chain, poem=poem_chain)
result = custom_map_chain.invoke({"joke": {"joke_topic": "bear"}, "poem": {"poem_topic": "sun"}})
In this modified version, the CustomRunnableParallel class inherits from RunnableParallel. The invoke method is overridden to accept a dictionary of inputs (inputs: Dict[str, Any]). Each key in this dictionary corresponds to a step (e.g., joke, poem), and the value is the specific input for that step. This allows you to provide different inputs for different chains.
