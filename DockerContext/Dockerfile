FROM python:3.10-buster

RUN set -ex \
    && git clone https://github.com/EricBoix/jj_doc_Zen_Flesh_Zen_Bones.git \
    # Rename archive for clear context when debugging/fixing in container
    && mv jj_doc_Zen_Flesh_Zen_Bones/ jj_doc_Zen_Flesh_Zen_Bones.git
    # FIXME: we should be pinning the version with e.g.
    # && cd jj_doc_Zen_Flesh_Zen_Bones.git \
    # && git checkout 444b50086c

WORKDIR /jj_doc_Zen_Flesh_Zen_Bones.git/Convert
RUN set -ex && pip install --root-user-action=ignore -r requirements.txt

RUN pip freeze --all

# Systematically run tests at build time (fail often, fail early)
RUN set -ex \
    && pip install --root-user-action=ignore -r requirements-dev.txt \
    && pytest test_main.py

ENTRYPOINT ["python3", "./main.py"]
