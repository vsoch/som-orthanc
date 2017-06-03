FROM jodogne/orthanc-plugins 
COPY orthanc.json /etc/orthanc/

# Re-download some of the missing resources
RUN mkdir /code

# Add some useful files to have in the image
ADD resources /code
