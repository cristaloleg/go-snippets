# Go snippets
Code snippets of Go

#### HTTP client

```go
func NewHTTPClient() *http.Client {
	return &http.Client{
		Timeout: 10 * time.Second,
		Transport: &http.Transport{
			Dial: (&net.Dialer{
				Timeout: 5 * time.Second,
			}).Dial,
			TLSHandshakeTimeout: 5 * time.Second,
		},
	}
}
```

#### Check connection

```go
import (
	"errors"
	"io"
	"net"
	"syscall"
)

func connCheck(c net.Conn) error {
	sconn, ok := c.(syscall.Conn)
	if !ok {
		return nil
	}
	
	rc, err := sconn.SyscallConn()
	if err != nil {
		return err
	}
	
	var n int
	var err error
	var buff [1]byte
	
	rerr := rc.Read(func(fd uintptr) bool {
		n, err = syscall.Read(int(fd), buff[:])
		return true
	})
	
	switch {
	case rerr != nil:
		return rerr
	case n == 0 && err == nil:
		return io.EOF
	case n > 0:
		return errors.New("unexpected read from socket")
	case err == syscall.EAGAIN || err == syscall.EWOULDBLOCK:
		return nil
	default:
		return err
	}
}
```
