
```dockerfile
FROM busybox:base
MAINTAINER base <1990frog@gmail.com>
RUN opkg-install bash
CMD ["/bin/bash"]
```