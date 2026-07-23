```python
@dataclass
class Provider:
    """The model seam, made explicit (ch-14). Any OpenAI-compatible endpoint.

    Swap the provider and nothing above this object changes.

    ``responder`` is an optional escape hatch: when set, ``chat()`` calls it
    instead of hitting the network. This is how the ``fake`` provider injects
    deterministic responses for offline tests. ``from_env`` always leaves it
    ``None`` — env-configured providers go over HTTP.
    """

    base_url: str
    model: str
    api_key: str = "x"
    responder: Callable[..., LLMResponse] | None = None

    @classmethod
    def from_env(cls) -> Provider:
        _load_dotenv()
        return cls(
            base_url=os.environ.get("LLM_BASE_URL", DEFAULT_BASE_URL),
            model=os.environ.get("LLM_MODEL", DEFAULT_MODEL),
            api_key=os.environ.get("LLM_API_KEY", DEFAULT_API_KEY),
        )

```

`@classmethod` makes a method inside class to be able to get class itself as argument `(cls)` 