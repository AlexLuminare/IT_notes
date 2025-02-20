`golang.org/x/sync/errgroup`

```go
func errGroup() {
	g, ctx := errgroup.WithContext(context.Background())

	g.Go(func() error {
		time.Sleep(time.Second)

		select {
		case <-ctx.Done():
			return nil
		default:
			fmt.Println("first started")
			time.Sleep(time.Second)
			return nil
		}
	})
	g.Go(func() error {
		fmt.Println("second started")
		return fmt.Errorf("unexpected error in request 2")
	})
	g.Go(func() error {
		select {
		case <-ctx.Done():
		default:
			fmt.Println("third started")
			time.Sleep(time.Second)
		}
		return nil
	})

	if err := g.Wait(); err != nil {
		fmt.Println(err)
	}
}
```