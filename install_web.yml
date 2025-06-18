---
- name: Install Apache and deploy web app
  hosts: all
  become: true
  tasks:
    - name: Update apt cache and install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Copy static index.html to Apache root directory
      copy:
        src: ../app/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: 0644

    - name: Ensure Apache service is running
      service:
        name: apache2
        state: started
        enabled: yes