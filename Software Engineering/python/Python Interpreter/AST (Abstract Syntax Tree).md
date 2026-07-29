just for now this is a where code will be parse into tree 

we use this to make a expression parser for llms

```python
import ast
import operator

_BINOPS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv,
    ast.Pow: operator.pow,
    ast.Mod: operator.mod,
}

def calculator(expr:str) -> str:
    """Evaluate a basic arithmetic expression safely"""

    def ev(node: ast.AST) -> float:
        if isinstance(node,ast.Constant) and isinstance(node.value,(int,float)):
            return node.value
        if isinstance(node,ast.BinOp) and type(node.op) in _BINOPS:
            return _BINOPS[type(node.op)](ev(node.left),ev(node.right))
        if isinstance(node,ast.UnaryOp) and isinstance(node.op,ast.USub):
            return -ev(node.operand)
        raise ValueError(f"Unsupported expression: {expr}")
  
    result = ev(ast.parse(expr,mode="eval").body)
    return str(int(result)) if isinstance(result,int) or result.is_integer() else str(result)
```
