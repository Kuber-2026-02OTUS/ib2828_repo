
Производим развертывание кластера версии 1.35.0 с помощью ansible
Выполняем последовательно три команды
ansible-playbook site.yml --tags commmon -b 
ansible-playbook site.yml --tags master -b 
ansible-playbook site.yml --tags worker -b 

Кластер развернут. Скриншот kubernetes-prod/images/k8s-1.35.jpg

===============================================================
Обновление будем проводить вручную

Выполняем обновление kubeadm на всех нодах
sudo dpkg -i /tmp/v_1.36/kubeadm_1.36.0-1.1_amd64.deb

1. Производим обновление на мастер ноде
    Выполняем обновление kubeadm
    sudo dpkg -i /tmp/v_1.36/kubeadm_1.36.0-1.1_amd64.deb
    
    Выполним план обновления
    kubeadm upgrade apply v1.36.0
    
    Обновим пакеты
    sudo dpkg -i /tmp/v_1.36/kubelet_1.36.0-1.1_amd64.deb
    sudo dpkg -i /tmp/v_1.36/kubectl_1.36.0-1.1_amd64.deb

2. Производим обновлние воркер нод (на примере одной ноды, остальные аналогично)
    
    Выводим ноду из нагрузки (команда выполняется на мастере)
    kubectl drain k-w1 --ignore-daemonsets --delete-emptydir-data
    
    Выполняем обновление kubeadm
    sudo dpkg -i /tmp/v_1.36/kubeadm_1.36.0-1.1_amd64.deb

    Выполняем обновление конфигурации ноды
    sudo kubeadm upgrade node

    Выполняем обновление kubelet и kubectl
    sudo dpkg -i /tmp/v_1.36/kubelet_1.36.0-1.1_amd64.deb
    sudo dpkg -i /tmp/v_1.36/kubectl_1.36.0-1.1_amd64.deb
    
    Перезапускаем kubelet:
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

    Возвращаем ноду под нагрузку
    kubectl uncordon k-w1

Кластер обновлен. Скриншот kubernetes-prod/images/k8s-1.36.jpg
