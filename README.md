# MultiFuseNet
A High‑Resolution Logging‑Data Generation Model Based on Multi‑Source Data Fusion

## Introduction
MultiFuseNet is a deep‑learning network that directly generates high‑resolution well‑logging curves from heterogeneous multi‑source geological data including mud‑logging, drilling, geochemical and engineering data, without requiring input of original well‑log curves. It achieves 8× resolution enhancement and outputs logging series with 0.125 m sampling interval.

Three core modules:
1. Variational Granularity Alignment (VGA) — Unify fine‑ / medium‑ / coarse‑grained heterogeneous data onto a uniform 1‑meter depth grid, implemented as pre‑processing step.
2. Multi‑Scale Mapping Module (MSM) — Multi‑scale feature pyramid, dynamic attention mechanism and residual fusion to exploit complementary information from multi‑granularity inputs.
3. Super‑Resolution Generation Module (SRG) — Contextual attention modeling plus CVAE‑based progressive three‑stage up‑sampling, producing 8 high‑resolution logging sample points.

## Workflow
1. Pre‑processing: Variational Granularity Alignment (VGA)
Call function variational_granularity_alignment() to project raw multi‑source data of different granularities onto uniform 1‑m depth coordinate.
VGA is a numpy‑based preprocessing function, not part of the TensorFlow computation graph.

2. Sample Segmentation (SS)
Slide window length L=90, step size =1 to generate training samples:
- Input sample shape: [N, 90, 31] (window length=90; feature dim=31 = 4 fine‑grained + 9 medium‑grained +18 coarse‑grained features)
- Label shape: [N, 8, 8] (8 output 0.125‑m logging points; 8 logging curves: GR,PE,SP,CNL,DEN,AC,RD,RS)

3. Model Training
model = MultiFuseNet()
model.fit(x_train, y_train, batch_size=128, epochs=1000, callbacks=training_callbacks)

4. Inference & Prediction
model = keras.models.load_model("multifusenet.keras", custom_objects={"combined_loss":combined_loss})
y_pred = model.predict(x_test)

## Hyper‑parameters (From Paper Table5)
Parameter             Value
Optimizer             Adam
Max epochs            1000
Early‑stop patience   100
Initial LR            0.001
LR schedule           Cosine decay
Weight decay          0.01
Batch size            128
Residual α            0.1
Focal loss γ          0.1
SSIM loss weight      0.005
Window length L       90
Sub‑window k          L/4

## Limitations & Notes
1. This is a data‑driven model with domain‑shift risk. Direct cross‑block deployment leads to performance degradation. Fine‑tuning with local well‑log samples is mandatory for new working areas.
2. Generated logs are statistical estimations. They shall NOT be the sole evidence for reserve evaluation or completion design. Expert geological interpretation is required.
3. Original field dataset is confidential and not publicly available. Users need to prepare self‑owned multi‑source geological and well‑log dataset.

## License
For academic research purposes only.
