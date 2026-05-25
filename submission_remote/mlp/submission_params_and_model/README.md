# Remote Submission Example

This directory contains a remote submission example for the benchmarking suite.

As part of the benchmarking process, all remote backends are expected to make their homomorphic encryption parameters explicit and reviewable, since these parameters directly affect security, performance, and comparability across submissions.
Ideally, such parameters should be reported automatically by the backend.

This doc includes a static description of the FHE parameters used by the remote-backend example, along with a brief description of the model architecture. While helpful for context, model architecture disclosure is not a requirement of the benchmarking process.

The parameters shown here are chosen for simplicity and reproducibility and are not intended to represent a security-hardened or final challenge submission.

---

## FHE parameters

The example uses a single set of homomorphic encryption parameters for all supported workload sizes:
```python
homomorphic_params = {
    "full_q_list_precision": (
        (61,),
          (45,),
    ),                    # modulus chain: ~106 bits total
    "n": 2 ** 12,         # polynomial degree
    "err_std": 3.19,      # standard deviation of the encryption noise
    "sk_hw": 0,           # secret key distribution: uniform in {-1, 0, 1}
    "g_base_bits": 4,     # decomposition base in evaluation key
    "pt_scale": 2 ** 20,  # initial plaintext scaling factor
}
```

---

## Model weights

The model weights are stored in:

- `digits_recognizer.pth`

They correspond to a simple two-layer fully connected network and are loaded as a standard PyTorch state dict:

```python
model = torch.load(
    f"{Path(__file__).parent}/digits_recognizer.pth",
    weights_only=True,
    map_location="cpu",
)

l1_weight = model["l1.weight"]
l2_weight = model["l2.weight"]
```

---

## Model architecture

The remote homomorphic inference follows the pipeline below:

```python
hom_pipeline = SequentialHomOp(
    ClientReshape((BATCH_SIZE, 28 * 28,)),      # preprocess: flatten 28x28 input image to a 784-dimensional vector
    HomLinear(l1_weight.shape, bias=False),     # first linear layer (no bias)
    HomSquare(),                                # square activation
    HomLinear(l2_weight.shape, bias=False),     # second linear layer (no bias)
    ClientReshape((BATCH_SIZE, 10,)),           # postprocess: reshape output to 10-class vector
)
```

Note that `SequentialHomOp`, `HomLinear`, etc., are internal classes. They are shown here to document the structure of the computation, not as runnable public code.

---

## Packing strategy and ring dimension

This example uses different ciphertext packing strategies depending on the batch size, which affects the effective ciphertext and evaluation key sizes.

**BATCH_SIZE == 1:**

  The ring dimension corresponds to the *feature dimension*.

  - The input vector of size 784 is packed into 2 ciphertexts, each with 512 slots (zero-padded).
  - Inner products with the weights matrices are computed via slot summation using log n rotation keys.
  - The activation requires a single relinearization key.

\
**BATCH_SIZE > 1:**

  The ring dimension corresponds to the *batch dimension*.

  - An input of shape `(B, 784)` is represented by 784 ciphertexts, each packing up to 512 batch elements (remaining slots are zero-padded).
  - Summation is performed across ciphertexts (non-ring dimension), so no rotation keys are required.
  - The activation still requires a single relinearization key.

---

## Example execution logs

### `batch_size = 1`

```
python3 harness/run_submission.py --remote 0

11:56:49 [harness] 1: Harness: MNIST Test dataset generation completed (elapsed: 7.3298s)
11:56:51 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.4867s)
         [harness] Cryptographic Context size: 1.2M
11:56:54 [harness] 2.2: Client: Key Generation completed (elapsed: 2.1759s)
         [harness] Client: Public and evaluation keys size: 46.0M
11:56:57 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 3.2013s)
11:56:57 [harness] 3: Server: (Encrypted) model preprocessing completed (elapsed: 0.0001s)
11:56:59 [harness] 4: Harness: Input generation for MNIST completed (elapsed: 2.5745s)
11:57:01 [harness] 5: Client: Input preprocessing completed (elapsed: 1.6053s)
11:57:03 [harness] 6: Client: Input encryption completed (elapsed: 1.6251s)
         [harness] Client: Encrypted input size: 128.1K
11:57:05 [harness] 7: Server: Encrypted ML Inference computation completed (elapsed: 1.8562s)
         [harness] Client: Encrypted results size: 128.1K
11:57:06 [harness] 8: Client: Result decryption completed (elapsed: 1.6598s)
11:57:06 [harness] 9: Client: Result postprocessing completed (elapsed: 0.0002s)
[harness] PASS  (expected=5, got=5)
[total latency] 24.5149s
```
- Public + evaluation keys size: 46 MB
- Encrypted input size: 128 KB
- Total inference latency: 210 ms
- Compute inference latency: 80 ms
-----

### `batch_size = 100`

```
python3 harness/run_submission.py --remote 1

20:12:38 [harness] 1: Harness: MNIST Test dataset generation completed (elapsed: 7.4943s)
20:12:41 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.921s)
         [harness] Cryptographic Context size: 835.0K
20:12:42 [harness] 2.2: Client: Key Generation completed (elapsed: 1.702s)
         [harness] Client: Public and evaluation keys size: 7.3M
20:12:45 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 2.2597s)
20:12:45 [harness] 3: Server: (Encrypted) model preprocessing completed (elapsed: 0.0002s)
20:12:47 [harness] 4: Harness: Input generation for MNIST completed (elapsed: 2.6214s)
20:12:49 [harness] 5: Client: Input preprocessing completed (elapsed: 1.6287s)
20:12:52 [harness] 6: Client: Input encryption completed (elapsed: 3.0083s)
         [harness] Client: Encrypted input size: 98.0M
20:12:55 [harness] 7: Server: Encrypted ML Inference computation completed (elapsed: 2.6501s)
         [harness] Client: Encrypted results size: 1.3M
20:12:57 [harness] 8: Client: Result decryption completed (elapsed: 1.9299s)
20:12:57 [harness] 9: Client: Result postprocessing completed (elapsed: 0.0002s)
20:12:59 [harness] 10.1: Harness: Run inference for harness plaintext model completed (elapsed: 2.5665s)
[harness] Encrypted model: 0.9400 (94/100 correct)
[harness] Harness model: 0.9900 (99/100 correct)
20:12:59 [harness] 10.2: Harness: Run quality check completed (elapsed: 0.0003s)
[total latency] 28.7825s
```
- Public + evaluation keys size: 7.3 MB
- Encrypted input size: 98 MB
- Total inference latency: 1.1 s
- Compute inference latency: 315 ms
-----

### `batch_size = 1000`

```
python3 harness/run_submission.py --remote 2

20:16:17 [harness] 1: Harness: MNIST Test dataset generation completed (elapsed: 7.3336s)
20:16:20 [harness] 2.1: Communication: Get cryptographic context completed (elapsed: 2.3775s)
         [harness] Cryptographic Context size: 835.0K
20:16:21 [harness] 2.2: Client: Key Generation completed (elapsed: 1.7088s)
         [harness] Client: Public and evaluation keys size: 7.3M
20:16:24 [harness] 2.3: Communication: Upload evaluation key completed (elapsed: 2.3162s)
20:16:24 [harness] 3: Server: (Encrypted) model preprocessing completed (elapsed: 0.0002s)
20:16:27 [harness] 4: Harness: Input generation for MNIST completed (elapsed: 3.1771s)
20:16:29 [harness] 5: Client: Input preprocessing completed (elapsed: 1.8815s)
20:16:32 [harness] 6: Client: Input encryption completed (elapsed: 3.0121s)
         [harness] Client: Encrypted input size: 98.0M
20:16:34 [harness] 7: Server: Encrypted ML Inference computation completed (elapsed: 2.6911s)
         [harness] Client: Encrypted results size: 1.3M
20:16:36 [harness] 8: Client: Result decryption completed (elapsed: 1.9127s)
20:16:36 [harness] 9: Client: Result postprocessing completed (elapsed: 0.0002s)
20:16:39 [harness] 10.1: Harness: Run inference for harness plaintext model completed (elapsed: 2.7296s)
[harness] Encrypted model: 0.9720 (972/1000 correct)
[harness] Harness model: 0.9810 (981/1000 correct)
20:16:39 [harness] 10.2: Harness: Run quality check completed (elapsed: 0.0007s)
[total latency] 29.1411s
```
- Public + evaluation keys size: 7.3 MB
- Encrypted input size: 98 MB
- Total inference latency: 0.9 s
- Compute inference latency: 314 ms

