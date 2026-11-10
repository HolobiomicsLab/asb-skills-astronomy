# Release log: astronomy v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:26:04Z | 342c3d50365593ece396fc76d43417091d3ea950 | `{"candidate_tree": "sha256:6ba1a6bfde5f00294424f22e4bf5cbfc81f046f52451168c6a18088dd98539a1", "gate_report": "3787b40377c5cdfc0712285e0049bd4277cb99c97c098d77cc7294ee17c86e72"}` |
| deposit-plan | 2026-11-09T14:26:05Z | 342c3d50365593ece396fc76d43417091d3ea950 | `{"archive_sha256": "ee47b4308e1c509eec0c1e04190ddb803d4e63062a3009732670f7e5271e5ff3"}` |
| deposit-draft | 2026-11-09T21:05:52Z | 342c3d50365593ece396fc76d43417091d3ea950 | `{"archive_sha256": "ee47b4308e1c509eec0c1e04190ddb803d4e63062a3009732670f7e5271e5ff3"}` |
| record-doi | 2026-11-10T10:47:43Z | 342c3d50365593ece396fc76d43417091d3ea950 | `{"CITATION.cff": "89f4870df6aa88b33bd459d5b436b4ba4a2245f775fd6263200bc1a249f459d5", "README.md": "a62f4ea5edd63f7eef2c7aa2ea3c5b06b9b191077617468ab96d244bfab60f41", "RELEASE.yaml": "07344cdaa552f74bb554acdcde60b6d08c47635a14918f22ff442cb88ee457cf", "marketplace.json": "a217156f9008a9aa10a0192e1d1e6513cf3fdd43a66d010054a64b2663d87c9e", "plugin.json": "d478d4bfe41696450ab029754d91ddfd9e6ee811327339d02146d5c4a4f1a820"}` |
| receipt | 2026-11-10T10:48:42Z | 342c3d50365593ece396fc76d43417091d3ea950 | `{"receipt": "4fb7483ff76d1aa5a0c498d542aae97211f445952a0c61eb41b6feacbec94c61"}` |
| manifest | 2026-11-10T10:51:14Z | f4a72101448e3ef8cd4f910608ce2264467c0384 | `{"catalogue_jsonld": "51ccccef8d8948c7a1825a042a4a3b5d3340e3f2bfb0c567a7f8400901446d2e", "collections_yaml": "9cd47782f45d39e1c9173cd7f64f98c5794ff83e158fc2dd3ef12e374618cd79"}` |
| tag | 2026-11-10T11:06:54Z | f4a72101448e3ef8cd4f910608ce2264467c0384 | `{}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093095; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
- OWNER: git tag astronomy-v1.0.0 f4a72101448e3ef8cd4f910608ce2264467c0384
- OWNER: git push origin astronomy-v1.0.0
- OWNER: git tag astronomy-v1.0.0 f4a72101448e3ef8cd4f910608ce2264467c0384
- OWNER: git push origin astronomy-v1.0.0
