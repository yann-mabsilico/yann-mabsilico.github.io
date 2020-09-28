---
layout: default
title: docker
---

{: .title}
DP2S - Installation

* TOC
{: toc}

# Root

    apt-get install libfreetype6-dev gfortran libffi-dev

## Cuda
Je ne sais pas si `nvidia-kernel-source` vient avec `nvidia-driver`:

	apt-get install nvidia-driver nvidia-kernel-source

Après ça, download `cuda-9.0` sur le site de nvidia, lance et installe `cuda` seulement, pas les drivers.

# Non-root
Installer `anaconda3` ou `miniconda3`.

	export PATH=/path/to/conda3/bin:$PATH
    conda create -n dp2s python=3.6
    source activate dp2s

    conda install -y pygpu=0.7.6 tensorflow=1.10.0 tensorflow-gpu=1.10.0

Virer les lignes `pygpu, tensorflow, tensorflow-gpu` de `installations/requirements.txt`.

    pip install -r installations/requirements.txt
	pip install fpdf

	# After installation of cuda 9.0.
	echo '/usr/local/cuda-9.0/lib64/' > /etc/ld.so.conf.d/cuda-9.0.conf
	ldconfig

	python deepprime2sec.py --config sample_configs/model_a.yaml

ET BOOM, `Killed` parce qu'il prend toute ma mémoire.

Le fichier de 12Go est au format numpy. Tu peux l'ouvrir sans le charger complètement avec :

	kikoo = numpy.load(filepath, mmap_mode='r')

puis le couper :

	numpy.save(filepath, kikoo[:1000])

Après ça, dp2s se lance mais ne trouve pas le GPU. Va dans `utility/training.py` et commente la ligne `CUDA_VISIBLE_DEVICES`.
Après ça :

	2020-08-06 17:56:56.432415: E tensorflow/stream_executor/cuda/cuda_dnn.cc:352] Could not create cudnn handle: CUDNN_STATUS_INTERNAL_ERROR

