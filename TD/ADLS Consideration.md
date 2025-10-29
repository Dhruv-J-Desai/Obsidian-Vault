```python
import socket, time, errno, select

def probe(host: str, port: int = 1433, timeout: float = 5.0):
    print(f"\n=== TCP probe {host}:{port} (timeout {timeout}s) ===")

    # Resolve DNS (you’ll also get the already-specified port back)
    try:
        infos = socket.getaddrinfo(host, port, type=socket.SOCK_STREAM)
    except Exception as e:
        print(f"❌ DNS resolution failed: {e}")
        return False

    # Try each address
    for family, socktype, proto, _, addr in infos:
        ip, p = addr[0], addr[1]
        print(f"\n--- Trying {ip}:{p} ---")
        s = socket.socket(family, socktype, proto)
        s.setblocking(False)
        start = time.time()
        try:
            rc = s.connect_ex((ip, p))
            # Immediate success
            if rc == 0:
                dt = (time.time() - start) * 1000
                print(f"✅ Connected in {dt:.1f} ms")
                s.close()
                return True

            # “Would block / in progress” → wait for writability
            if rc in (errno.EINPROGRESS, errno.EWOULDBLOCK, getattr(errno, "WSAEWOULDBLOCK", 10035)):
                # Wait until the socket is writable or timeout
                r, w, e = select.select([], [s], [s], timeout)
                if s in w:
                    err = s.getsockopt(socket.SOL_SOCKET, socket.SO_ERROR)
                    dt = (time.time() - start) * 1000
                    if err == 0:
                        print(f"✅ Connected in {dt:.1f} ms")
                        s.close()
                        return True
                    else:
                        print(f"❌ Connect completed with error SO_ERROR={err}")
                else:
                    print("❌ Timed out waiting for TCP handshake (likely filtered by firewall or path issue)")
            else:
                # Hard immediate error
                print(f"❌ connect_ex returned {rc} ({errno.errorcode.get(rc, 'UNKNOWN')}) immediately")

        finally:
            s.close()

    print("❌ All addresses failed")
    return False

# Example:
probe("your-sql-server-host", 1433)

```