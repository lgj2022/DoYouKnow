From python:3.9

ENV PYTHONUNBUFFERED 1

RUN apt-get -y update

RUN apt-get -y install vim

COPY ./ /var/jenkins_home/workspace/deploytest/Backend/Django

WORKDIR /var/jenkins_home/workspace/deploytest/Backend/Django


RUN pip3 install --upgrade pip

RUN pip3 install -r requirements.txt

EXPOSE 8000

CMD ["python", "manage.py", "makemigrations"]

CMD ["python", "manage.py", "migrate"]

CMD ["python", "manage.py","runserver","0.0.0.0:8000"]
