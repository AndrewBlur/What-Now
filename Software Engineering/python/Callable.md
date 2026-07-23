```python
from typing import Callable

class LLMResponse:
    def __init__(self, text: str):
        self.text = text

def my_responder(prompt: str, temperature: float) -> LLMResponse:
    return LLMResponse(f"Response to: {prompt}")

responder: Callable[..., LLMResponse] | None = my_responder
```
anything that is callable as function can be given as Callable type