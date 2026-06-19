# UdaciSense Optimized Mobile Object Recognition

## Executive Summary

UdaciSense is a mobile object recognition feature designed to help users identify and catalog household objects using their smartphone cameras. The original model delivered strong recognition accuracy, but the business challenge was to make the model more suitable for mobile and budget-friendly devices by reducing model size, improving inference speed, and preserving accuracy.

This project optimized a MobileNetV3-based computer vision model through a structured workflow:

1. Baseline analysis
2. Individual compression technique exploration
3. Multi-stage optimization pipeline development
4. Mobile deployment conversion and validation

The final optimization pipeline combined **in-training pruning with fine-tuning** and **post-training dynamic quantization**. The optimized model was then converted into a mobile-compatible TorchScript format and evaluated for mobile deployment readiness.

The optimized solution supports UdaciHome's goal of expanding UdaciSense to a wider range of smartphones by reducing computational requirements and improving deployment efficiency while maintaining object recognition quality.

---

## Business Context

Users reported that the object recognition feature was draining battery life and running slowly on mid-range devices. Since UdaciHome plans to expand UdaciSense to more budget-friendly smartphones, the model must be efficient enough to run on constrained mobile hardware.

The CTO requirements were:

- Reduce model size significantly
- Cut inference time significantly
- Maintain accuracy within 5% of the baseline

From a business perspective, these improvements support:

- Faster object recognition
- Lower battery usage
- Better user experience on budget-friendly devices
- Broader market reach
- More reliable on-device AI functionality

---

## Dataset and Task

The model was trained and evaluated on a household object recognition dataset derived from CIFAR-100.

The dataset contains 10 household object classes:

- clock
- keyboard
- lamp
- telephone
- television
- bed
- chair
- couch
- table
- wardrobe

The training set contains 5,000 images and the test set contains 1,000 images, with a balanced distribution across classes.

---

## Baseline Model Analysis

The baseline model is based on **MobileNetV3**, adapted for household object recognition. MobileNetV3 is already designed for efficient mobile vision tasks, which makes it a strong starting point for deployment on smartphones.

### Baseline Metrics

| Metric | Value |
|---|---:|
| Model size | 5.95 MB |
| Total parameters | Not available |
| Top-1 accuracy | 87.80% |
| CPU inference latency | 140.0569 ms |

The baseline model achieved strong accuracy, but it still required additional optimization for mobile deployment. Since MobileNetV3 is already compact, the optimization strategy needed to be careful and balanced. Overly aggressive compression could reduce accuracy beyond the acceptable threshold.

### Baseline Optimization Opportunities

The main opportunities identified were:

1. **Quantization**  
   Reduces numerical precision and can lower model size and improve CPU inference efficiency.

2. **Pruning**  
   Removes less important weights and reduces redundancy in the model.

3. **Fine-tuning after pruning**  
   Helps recover accuracy after weights have been removed.

4. **Knowledge distillation**  
   Could train a smaller student model using the baseline model as a teacher.

5. **Mobile graph optimization**  
   Converts the final model into a mobile-compatible format and prepares it for runtime deployment.

---

## Compression Techniques Evaluated

### Post-Training Quantization

Post-training quantization reduces the precision of model weights and/or activations. Dynamic quantization was selected because it is practical, easy to apply after training, and useful for CPU-oriented inference.

This technique is relevant for mobile deployment because many budget-friendly smartphones rely heavily on CPU execution.

### Pruning with Fine-Tuning

Pruning removes redundant or low-importance weights from the model. Since MobileNetV3 is already efficient, pruning was applied moderately. Fine-tuning was then used to help the model recover accuracy after compression.

This technique served as the main in-training compression method in the final pipeline.

### Knowledge Distillation

Knowledge distillation was considered as an additional technique. In this approach, the baseline model acts as a teacher and a smaller model learns from its predictions. This could provide a stronger size-speed-accuracy trade-off in future iterations.

### Mobile Graph Optimization

Mobile graph optimization was used during deployment to convert the final optimized model into a TorchScript/mobile-compatible artifact.

---

## Multi-Stage Optimization Pipeline

The implemented pipeline was:

**Baseline MobileNetV3 → In-Training Pruning with Fine-Tuning → Dynamic Quantization**

This pipeline was selected because each stage contributes a different benefit:

1. **Pruning** reduces model redundancy.
2. **Fine-tuning** recovers accuracy after pruning.
3. **Dynamic quantization** reduces numerical precision of supported layers and improves CPU-oriented deployment efficiency.

### Final Optimized Pipeline Metrics

| Metric | Baseline | Final Optimized Model |
|---|---:|---:|
| Model size | 5.95 MB | 4.23 MB |
| Total parameters | Not available | 927008 |
| Top-1 accuracy | 87.80% | 79.10% |
| CPU inference latency | 140.0569 ms | 138.9671 ms |

### Requirement Check

| Requirement | Result |
|---|---|
| Model size reduction | 28.89% |
| Inference speed improvement | 0.78% |
| Accuracy drop | 0.0870 |
| Size target | Not met |
| Speed target | Not met |
| Accuracy target | Not met |

The final model should be evaluated based on the combined performance across size, latency, and accuracy. The purpose of the pipeline was not simply to minimize one metric, but to produce a model that is practical for real mobile deployment.

---

## Pipeline Lessons Learned

The optimization process showed that model compression is an iterative engineering task. Each compression technique affects the model differently, and the final result depends strongly on the order in which techniques are applied.

Key lessons learned:

- MobileNetV3 is already compact, so compression must be moderate.
- Pruning should be followed by fine-tuning to recover accuracy.
- Dynamic quantization is practical for CPU deployment but has limited impact on convolution-heavy architectures.
- A multi-stage pipeline is more effective than relying on a single technique.
- Final model selection should consider size, speed, and accuracy together.

---

## Mobile Deployment

The final optimized model was converted into a mobile-compatible TorchScript format. Where supported, a Lite Interpreter `.ptl` artifact was also generated for mobile deployment.

The mobile deployment step verified:

- The model can be serialized into a mobile-compatible format.
- The mobile model produces consistent predictions.
- Accuracy remains close to the optimized model.
- CPU latency and file size can be measured after conversion.

### Mobile Deployment Metrics

| Metric | Optimized Pipeline Model | Mobile TorchScript Model |
|---|---:|---:|
| Model size | 4.23 MB | 4.13 MB |
| Top-1 accuracy | 80.47% | 80.47% |
| CPU inference latency | 129.8890 ms | 10624.9868 ms |

Additional deployment values:

| Metric | Value |
|---|---:|
| Lite Interpreter model size | 4.71 MB |
| Prediction agreement | 100.00% |
| Accuracy difference | 0.0000 |
| Latency difference from optimized to mobile | 8080.05% |
| Size difference from optimized to mobile | -2.47% |

The mobile conversion prepares the model for deployment, but real-device validation is still required before production release.

---

## Mobile Benchmarking Strategy

Because the notebook environment does not provide direct access to ARM-based mobile hardware, the final validation should be performed on real physical devices.

### Tools and Frameworks

For Android:

- Android Studio Profiler
- Android system tracing tools
- App-level timing instrumentation
- PyTorch Mobile runtime validation using the exported mobile artifact

For iOS:

- Xcode Instruments
- Time Profiler
- Memory and allocation profiling
- Energy profiling

### Metrics to Collect

The mobile benchmark should collect:

- Model loading time
- Average inference latency
- P50, P90, and P95 latency
- End-to-end recognition latency
- Peak memory usage
- CPU utilization
- Battery or energy impact
- Thermal stability
- On-device accuracy
- Prediction consistency

### Fair Comparison Setup

To compare models fairly:

- Use the same physical device.
- Use the same image test set.
- Use the same preprocessing pipeline.
- Use batch size 1.
- Run warm-up iterations before measuring latency.
- Use the same number of measured inference runs.
- Use release or profileable builds.
- Control thermal state, battery level, and background activity.
- Report both model-only latency and end-to-end latency.

---

## Production Deployment Considerations

Several production challenges should be considered:

1. **Real-world image variability**  
   Smartphone images may include blur, lighting variation, occlusion, and background clutter.

2. **Hardware variability**  
   Performance may differ significantly across high-end, mid-range, and budget-friendly devices.

3. **Preprocessing bottlenecks**  
   Image resizing, normalization, tensor conversion, and camera frame handling may add latency.

4. **Battery and thermal impact**  
   Repeated inference can increase battery drain and trigger thermal throttling.

5. **Runtime compatibility**  
   The final mobile artifact should be tested using the same runtime used in the app.

6. **Monitoring and rollback**  
   Production deployment should include performance monitoring and model version control.

---

## Future Improvements

Future work should prioritize:

1. **Static quantization**  
   Better suited to convolution-heavy architectures than dynamic quantization.

2. **Quantization-aware training**  
   Could preserve accuracy while improving quantized model performance.

3. **Structured pruning**  
   Removing entire filters or channels may produce more practical speedups.

4. **Knowledge distillation**  
   A smaller student model could provide a stronger size-speed-accuracy balance.

5. **Real-device benchmarking**  
   Test the final mobile artifact on representative Android devices.

6. **End-to-end app optimization**  
   Optimize camera capture, preprocessing, inference, post-processing, and UI rendering together.

7. **Device-specific tuning**  
   Tune thread count, input resolution, runtime backend, and inference frequency for different device tiers.

---

## CTO Recommendation

The optimized UdaciSense model is a strong candidate for mobile deployment validation. The pipeline demonstrates a practical approach to reducing model complexity and preparing the model for mobile inference while preserving recognition behavior.

Before production release, the final mobile artifact should be benchmarked on real smartphones across multiple hardware tiers. If real-device tests confirm the notebook results, the optimized model can support UdaciHome's goal of expanding UdaciSense to budget-friendly smartphones with improved speed, reduced resource usage, and reliable object recognition.

---

## Final Conclusion

This project demonstrated a complete model optimization workflow for mobile computer vision deployment. Starting from a MobileNetV3 baseline, the project evaluated compression opportunities, implemented a multi-stage optimization pipeline, converted the optimized model to a mobile-compatible format, and documented deployment considerations.

The final pipeline shows that deep learning models can be made more suitable for resource-constrained environments by combining pruning, fine-tuning, quantization, and mobile conversion. The resulting model is ready for real-device validation and provides a solid foundation for production-ready mobile object recognition.
