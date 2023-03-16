# Data buffers and Codecs
sping-core provides a set of abstractions to work with various byte buffer APIs:
- `DataBufferFactory`: abstrcts the creation of data buffer
- `DataBuffer`: represents a byte buffer, which maybe `pooled`
- `DataBufferUtils`: utility methods for data buffers
- `Codecs` decode and endcode data buffer streams into higher level objects.
  
## 8.1 DataBufferFactory
 Used to creata data buffers in one of two ways.
 - Allocate a new data buffer, specifying capacity upfront.
 - Wrap an exsting `byte[]` or `java.nio.ByeBuffer`.

## 8.2 DataBuffer
Offers similar perations as `java.nio.ByteBuffer`, brings a few additional benefits of which are inspired by Netty `ByteBuf` 
- Read and write with independent positions (No need call flip to alternate between read and write)
- Capacity expanded on demand as with SpringBuilder
- Pooled buffer and reference counting via PooledDataBuffer.
- View a buffer as java.nio.ByteBuffer, InputStream or OutputStream.
- Determine the index or last index for a given byte.

## 8.3 PooledDataBuffer
 - Java `ByteBuffer` can direct or non direct. Direct buffers may reside outside the Java heap. that makes direct buffers particularly useful for receiving and sending data over a socket - but also more expensive to create and release => It lead to pool idea.
 - `PooledDataBuffer` is an extension of `DataBuffer`, help with refernce counting which is essential for byte buffer pooling. `PooledDataBuffer` is allocated then the reference count is at 1. Call `retain()` increment the count, `release()`  decreament, al long as the count is above 0, the buffer is guaranteed not to be released, count = 0, pooled buffer can be released - release is mean memory for the buffer is returnted to the memory pool (Most cases it's better to use method in `DataBufferUtils` to release or retain)

## 8.4 DataBufferUtils
- Join a stream of data buffers into a single buffer possibly with zero copy // ko hieu gi
- Turn `InputStream` or NIO `Channel` into `Flux<DataBuffer>` and vice versa a `Publisher<DataBuffer>` into `OutputStream` or NIO `Channel`
- Method to release or retain a `DataBuffer` if the buffer is an instance of `PooledDataBuffer`
- Skip or take from stream of bytes until a specific byte count.

# 8.5 Codecs
`org.springframework.core.codec` package
- Encoder to endcode `Publisher<T>` into stream of data buffers.
- Decoder to decode `Publisher<DataBuffer>` into a stream of higher level objects.

# 8.6 Using DataBuffer.