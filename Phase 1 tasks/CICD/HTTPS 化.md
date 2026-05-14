* sửa Gitlab server nhận https từ alb về:
sudo editor /etc/gitlab/gitlab.rb
```
# Đặt External URL thành HTTPS với subdomain của bạn (ví dụ: gitlab.example.com)
external_url 'https://gitlab.example.com'

# Tắt HTTPS listener của NGINX GitLab vì ALB đã terminate SSL
nginx['listen_https'] = false
nginx['listen_port'] = 80  # Chỉ listen HTTP trên port 80

# Cấu hình proxy headers để GitLab nhận biết protocol thực tế từ ALB
nginx['proxy_set_headers'] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}

# Cấu hình real_ip để lấy IP client thực từ ALB (thay bằng subnet VPC hoặc IP range của ALB)
nginx['real_ip_trusted_addresses'] = ['10.0.0.0/8']  # Ví dụ subnet AWS VPC
nginx['real_ip_header'] = 'X-Forwarded-For'
nginx['real_ip_recursive'] = 'on'

# Tùy chọn: Redirect HTTP sang HTTPS (nếu cần)
nginx['redirect_http_to_https'] = true
```
Áp dụng thay đổi:
```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```
