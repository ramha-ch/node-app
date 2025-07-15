# node-app

name: Build and Deploy Node App with Docker

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Log in to Docker Hub
      run: |
        echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

    - name: Build Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/node-app:latest .

    - name: Push Docker image to Docker Hub
      run: |
        docker push ${{ secrets.DOCKER_USERNAME }}/node-app:latest

    - name: Setup SSH and Deploy on EC2
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.EC2_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H ${{ secrets.EC2_HOST }} >> ~/.ssh/known_hosts

        ssh ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} << EOF
          docker pull ${{ secrets.DOCKER_USERNAME }}/node-app:latest

          docker stop node-app || true
          docker rm node-app || true

          docker run -d --name node-app -p 3000:3000 ${{ secrets.DOCKER_USERNAME }}/node-app:latest
        EOF
