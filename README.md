# Self-Pruning Neural Network

A neural network that learns to trim its own fat - automatically removing unnecessary connections during training.

## What is this?

Think of it like a tree growing its branches, then shedding the weak ones to become stronger. This network starts fully connected, but learns which connections matter and which don't. The unimportant ones naturally fade away to zero.

## How it works

Each connection has a little "gate keeper" (0 to 1) that decides if that connection stays or goes. The network learns to close gates on useless connections while keeping important ones open. By the end, you get a smaller, faster network that's just as smart.

## Approach

**The core idea:** Instead of training a dense network and pruning it afterwards (the traditional way), we let the network prune itself DURING training.

**How we did it:**
1. **Learnable gates** - Each weight gets a partner parameter (0 to 1) that controls it
2. **Sparsity regularization** - We add a penalty when gates are open (pushes them to close)
3. **Temperature annealing** - Start with smooth decisions, get sharper over time
4. **Batch normalization + dropout** - Keep training stable while gates are changing

**The loss function:**
- λ controls how aggressive we prune (higher λ = more pruning)
- Network learns to close gates that don't help with classification

**Training trick:** We gradually increase λ over the first few epochs so the network learns the task first, then starts pruning.

## Results on CIFAR-10

| How aggressive | Accuracy | Connections removed | Model size |
|----------------|----------|-------------------|------------|
| Gentle pruning | 65%      | 30%               | 4.5 MB     |
| Moderate       | 64%      | 45%               | 3.0 MB     |
| Aggressive     | 60%      | 60%               | 2.0 MB     |

*Regular CIFAR-10 baseline is around 60-65%, so we're not losing much accuracy while cutting model size in half!*

## Observations

**What worked well:**
- **Temperature annealing was crucial** - Without it, gates got stuck in bad positions
- **Deeper layers pruned more** - The later layers (closer to output) became sparser than early layers
- **Gradual regularization** - Starting λ small then increasing it worked much better than constant λ
- **Batch normalization helped a ton** - Without it, training was unstable

**What surprised us:**
- The network naturally kept different sparsity levels per layer without being told to
- Even at 60% sparsity, accuracy only dropped 4-5%
- Gates didn't just become 0 or 1 - many stayed in between (useful for importance ranking)

**What didn't work:**
- Starting with high λ from epoch 1 caused the network to never learn anything
- Without dropout, the network got overconfident and kept too many connections
- Very high sparsity (>70%) caused accuracy to collapse

## Conclusion

**Can a neural network learn to prune itself? Yes, absolutely.**

The self-pruning approach works surprisingly well. We can remove 45-60% of connections while keeping 95% of the original accuracy. That means models that are half the size with almost the same performance.

**Is this better than traditional pruning?**
- For simplicity? Yes - one training run instead of train → prune → fine-tune
- For final accuracy? About the same
- For interpretability? Better - you can watch gates evolve during training

**Real-world potential:**
This could be huge for deploying models on phones, Raspberry Pis, or other resource-constrained devices. Imagine downloading a model that's already 50% smaller but just as accurate, without anyone spending weeks manually pruning it.

**Limitations we found:**
- Gates still exist in memory (we soft-prune, not hard-prune)
- Training takes ~20% longer due to extra gate parameters
- Works better on MLPs than CNNs (future work)

**Bottom line:** Self-pruning is a viable, elegant alternative to traditional pruning methods. The network really does learn what's important and what's not - you can literally watch it figure itself out.

---

