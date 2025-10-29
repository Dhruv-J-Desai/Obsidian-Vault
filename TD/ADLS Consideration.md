import socket, time, errno, traceback

def explain_errno(code: int) -> str:
    names = {v: k for k, v in errno.__dict__.items() if isinstance(v, int)}
    return names.get(code, "UNKNOWN")

def tcp_probe(host: str, port: int = 1433, timeout: float = 5.0):
    print(f"\n=== SQL TCP Probe: {host}:{port} (timeout {timeout}s) ===")

    # DNS
    t0 = time.time()
    try:
        infos = socket.getaddrinfo(host, port, type=socket.SOCK_STREAM)
        addrs = list({(ai[4][0], ai[4][1]) for ai in infos})
        t_dns = (time.time() - t0) * 1000
        print(f"✅ DNS resolved in {t_dns:.1f} ms -> {', '.join(ip for ip,_ in addrs)}")
    except Exception as e:
        print("❌ DNS resolution failed")
        traceback.print_exc()
        return

    # Try each resolved address
    for ip, p in addrs:
        print(f"\n--- Trying {ip}:{p} ---")
        s = socket.socket(socket.AF_INET if ':' not in ip else socket.AF_INET6, socket.SOCK_STREAM)
        s.settimeout(timeout)
        t1 = time.time()
        rc = s.connect_ex((ip, p))  # returns errno-style code (0 = success)
        dt = (time.time() - t1) * 1000
        if rc == 0:
            print(f"✅ TCP connected in {dt:.1f} ms")
            s.close()
            return
        else:
            name = explain_errno(rc)
            print(f"❌ TCP connect_ex rc={rc} ({name}) after {dt:.1f} ms")
            # Common hints
            if rc in (errno.ETIMEDOUT,):
                print("Hint: Timeout → firewall dropping packets, wrong network/VPN, or host not reachable on this path.")
            elif rc in (errno.ECONNREFUSED,):
                print("Hint: Connection refused → host reachable but nothing is listening on that port (service down/wrong port).")
            elif rc in (errno.EHOSTUNREACH, errno.ENETUNREACH):
                print("Hint: Host/Network unreachable → routing/VPN/VNet/peering issue.")
            elif rc in (errno.EADDRNOTAVAIL,):
                print("Hint: Local address not available → local networking misconfig (rare).")
            else:
                print("Hint: See OS firewall rules, security groups/NSGs, or server listener config.")
        s.close()

# Example:
tcp_probe("your-sql-server-host", 1433)
