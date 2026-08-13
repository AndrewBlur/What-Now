Model Compression:

## Squeeze Nets

`10 × 10 × 256 → 10 × 10 × 512` using a 3×3 convolution


```
3 × 3     → kernel size
256       → input channels
512       → output channels / filters
```

So:

3×3×256×512=1,179,648

So **1,179,648 weights**.

### Compare with our Fire Module

```
                10×10×256
                     │
                 SQUEEZE
                 256 → 64
                     │
                     ▼
                10×10×64
                     │
          ┌──────────┴──────────┐
          │                     │
       1×1 conv              3×3 conv
        64 → 256               64 → 256
          │                     │
          ▼                     ▼
     10×10×256            10×10×256
          │                     │
          └──────────┬──────────┘
                     │
                 CONCATENATE
                     │
                     ▼
                10×10×512
```

```
Squeeze:
1 × 1 × 256 × 64 = 16,384

Expand 1×1:
1 × 1 × 64 × 256 = 16,384

Expand 3×3:
3 × 3 × 64 × 256 = 147,456
```

Total:

16,384+16,384+147,456=180,224

So:

```
Direct 3×3:     1,179,648
Fire Module:      180,224
                ─────────
                 ~6.5× fewer
```

**That's the main reason SqueezeNet works.** The expensive 3×3 convolution operates on only **64 channels**, instead of 256.

A normal convolution does:

```
256 channels
      │
      │ 3×3
      ▼
512 channels
```

A Fire Module does:

```
256 channels
      │
      │ 1×1
      ▼
64 channels
      │
   ┌──┴──┐
   │     │
  1×1   3×3
   │     │
   ▼     ▼
 256    256
   │     │
   └──┬──┘
      ▼
    512
```

The **64 channels are not simply throwing away 192 channels**.

The 1×1 convolution **learns 64 new combinations of the original 256 channels**.

For example:

```
Original:
[x1, x2, x3, ..., x256]

1×1 filter #1:
y1 = w1x1 + w2x2 + ... + w256x256

1×1 filter #2:
y2 = w1x1 + w2x2 + ... + w256x256

...

64 filters → 64 learned features
```

So the network learns:

> "Which combinations of these 256 features are useful enough to pass into the expensive 3×3 operation?"

Then the 3×3 convolution learns **spatial patterns** from those 64 learned features.

### But there IS a tradeoff

You're giving the network a narrower bottleneck:

```
256 → 64
```

So it has **less representational capacity** than the full 256-channel 3×3 convolution.

SqueezeNet relies on the fact that this reduced representation can still preserve enough useful information while dramatically reducing computation.

##  Low Rank Factorization




	
	Low Rank Factorization
	mobilenets - pointwise cnn
	quantization
	deep compression

Model Optimization:
	- loop tiling 
	- vectorization
	- operator fusion
	- parallization
	- graph optimzation

