# Setting Up API Access on Oracle Cloud Instances

This document provides a step-by-step guide for making your APIs accessible from external clients (like Postman) when running on Oracle Cloud Infrastructure.

## 1. Docker Configuration

Ensure your Docker Compose file correctly maps container ports to host ports:

```yaml
services:
  your-api-service:
    # other configuration...
    ports:
      - "8080:8080"  # Maps host port 8080 to container port 8080
```

## 2. Verify Docker Container Status

Confirm containers are running and ports are properly mapped:

```bash
docker ps
```

Look for output showing port mappings like: `0.0.0.0:8080->8080/tcp`

## 3. Check Application Configuration

Ensure your application is listening on all interfaces (0.0.0.0), not just localhost:

```bash
docker exec <container_name> netstat -tulpn | grep <port>
```

You should see something like: `tcp 0 0 0.0.0.0:8080 0.0.0.0:* LISTEN` or `tcp 0 0 :::8080 :::* LISTEN`

## 4. Configure OS-Level Firewall (iptables)

Add a rule to allow incoming traffic on your API port:

```bash
sudo iptables -I INPUT 5 -p tcp --dport 8080 -j ACCEPT
```

Make the rule persistent:

```bash
sudo netfilter-persistent save
# or
sudo service iptables save
```

## 5. Configure Oracle Cloud VCN Security Lists (CRITICAL)

This is often the missing step when APIs aren't accessible:

1. Navigate to the Oracle Cloud Console
2. Go to: Hamburger menu (≡) > Networking > Virtual Cloud Networks
3. Select your VCN
4. Click on "Security Lists" in the left sidebar
5. Select the security list associated with your subnet
6. Click "Add Ingress Rules"
7. Add a rule with these settings:
   - Source Type: CIDR
   - Source CIDR: 0.0.0.0/0 (for public access) or your specific IP range
   - IP Protocol: TCP
   - Destination Port Range: 8080 (or your API port)
   - Description: API access

## 6. Testing Access

Test locally on the instance:

```bash
curl localhost:8080/api
```

Test externally using Postman with:
```
http://<server-public-ip>:8080/api
```

## Troubleshooting

If you still can't access your API:

1. Check if your instance has a public IP address
2. Verify all security groups/lists associated with the instance
3. Ensure no additional firewall software is running on the instance
4. Check application logs for binding errors
5. Try accessing a different port to isolate if it's an application-specific issue

Remember to follow security best practices by only exposing necessary ports and considering implementing API authentication.
