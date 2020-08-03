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

