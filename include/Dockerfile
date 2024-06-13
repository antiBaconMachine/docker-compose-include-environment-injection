FROM bash
ARG TARGET_MESSAGE="default from dockerfile"
ENV TARGET_MESSAGE=${TARGET_MESSAGE}
CMD [ "bash", "-c", "env && [[ $MESSAGE == $TARGET_MESSAGE ]]" ]