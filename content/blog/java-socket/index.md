---
title: "Kết nối Socket trong Java"
date: 2025-09-21
tags: ["Java", "Networking"]
featuredImage: "/images/featured.jpg"
---

Trong lập trình mạng bằng Java, `Socket` là công cụ giúp kết nối giữa client và server.  
Bằng cách sử dụng **java.net.Socket** và **java.net.ServerSocket**, ta có thể xây dựng các ứng dụng chat, truyền file hoặc HTTP server đơn giản.

{{< figure src="socket-diagram.png" title="Mô hình Client-Server cơ bản trong lập trình Socket" >}}

Ở mức thấp hơn, kết nối TCP được thiết lập qua quá trình **bắt tay 3 bước (TCP Handshake)**.  
Điều này đảm bảo client và server đồng bộ trạng thái trước khi truyền dữ liệu.

{{< figure src="tcp.png" title="Cơ chế bắt tay 3 bước TCP (Three-Way Handshake)" >}}
