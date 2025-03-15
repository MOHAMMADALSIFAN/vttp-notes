# File retrieval
## Multiple endpoints for different data
Would be the cleanest approach architecturally.
```java
@RestController
@RequestMapping("/api/files")
public class FileController {

    private final JdbcTemplate template;
    
    public FileController(JdbcTemplate template) {
        this.template = template;
    }
    
    // Endpoint for file metadata
    @GetMapping("/{id}/metadata")
    public ResponseEntity<FileMetadata> getFileMetadata(@PathVariable Long id) {
        Optional<FileMetadata> metadata = template.query("select id, name, media_type, creation_date, author, description from files where id = ?", 
            (ResultSet rs) -> {
                if (!rs.next())
                    return Optional.empty();
                
                FileMetadata meta = new FileMetadata();
                meta.setId(rs.getLong("id"));
                meta.setName(rs.getString("name"));
                meta.setContentType(rs.getString("media_type"));
                meta.setCreationDate(rs.getTimestamp("creation_date").toLocalDateTime());
                meta.setAuthor(rs.getString("author"));
                meta.setDescription(rs.getString("description"));
                
                return Optional.of(meta);
            }, id);
        
        return metadata.map(ResponseEntity::ok)
                      .orElse(ResponseEntity.notFound().build());
    }
    
    // Endpoint for file content
    @GetMapping("/{id}/content")
    public ResponseEntity<byte[]> getFileContent(@PathVariable Long id) {
        // Your existing file retrieval logic
        Optional<FileData> opt = template.query("select * from files where id = ?", 
            (ResultSet rs) -> {
                if (!rs.next())
                    return Optional.empty();
                FileData file = new FileData();
                file.setName(rs.getString("name"));
                file.setContentType(rs.getString("media_type"));
                file.setContent(rs.getBytes("content"));
            
                return Optional.of(file);
            }, id);
        
        if (opt.isPresent()) {
            FileData file = opt.get();
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.parseMediaType(file.getContentType()));
            headers.setContentDispositionFormData("inline", file.getName());
            
            return ResponseEntity.ok()
                .headers(headers)
                .body(file.getContent());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

```ts
@Injectable({
	providedIn: 'root'
})
export class FileService {
	http = inject(HttpClient);
	sanitizer = inject(DomSanitizer);

	// Get file metadata
	getFileMetaData(id: number): Observable<FileMetadata> {
		return this.http.get<FileMetadata>(`/api/files/${id}/metadata`);
	}

	// Get file content
	getFileContent(id: number): Observable<Blob>  {
		return this.http.get(`/api/files/${id}/content`, { responseType: 'blob' });
	}

	// Combine both requests if needed
	getCompleteFile(id: number): Observable<{metadata: FileMetadata, content: SafeUrl}> {
		return forkJoin({
			metadata: this.getFileMetadata(id),
			contentBlob: this.getFileContent(id)
		}).pipe(
			map(result => {
				const url = URL.createObjectURL(result.contentBlob);
				return {
					metadata: result.metadata,
					content: this.snaitizer.bypassSecurityTrustUrl(url)
				};
			})
		);
	}
}
```

## Base64 Encoding
```java
FileData file = ...

FileResponse response = new FileResponse();
response.setId(file.getId());
response.setName(file.getName());
...
setContentType()
setCreationDate()
setAuthor()
setDescription()
...

response.setBase64Content(Base64.getEncoder()
	.encodeToString(file.getContent()));

return ResponseEntity.ok(response);
```
```ts
getFileWithMetadata(id: number): Observable<FileWithMetadata> {
	return this.http.get<FileWithMetadata>(`/api/files/${id}`).pipe(
		map(response => {
			const binaryString = window.atob(response.bas64Content);
			const bytes = new Unit8Array(binaryString.length);
			for (let i = 0; i < binaryString.length; i++) {
				bytes[i] = binaryString.charCodeAt(i);
			}
			const blob = new Blob([bytes], {type: response.contentType });
			const url = URL.createObjectURL(blob);

			response.safeUrl = this.sanitizer.bypassSecurityTrustUrl(url);
			return response;
		})
	)
}
```

## Display images
### Using Object URLs with DomSanitizer
```ts
imageUrl!: SafeUrl;

ngOnInit() {
	this.fileService.getFileContent(123).subscribe(blob => {
		const unsafeUrl = URL.createObjectUrl(blob);
		this.imageUrl = this.sanitizer.bypassSecurityTrustUrl(unsafeUrl);
	});
}

ngOnDestroy() {
	if (this.imageUrl) {
		URL.revokeObjectUrl(this.imageUrl.toString());
	}
}
```
```html
<img [src]="imageUrl">
```

### Using Base64 Encoding directly
```ts
base64Image: SafeUrl;

ngOnInit() {
	this.fileService.getFileWithMetadata(123).subscribe(response => {
		const dataUrl = `data:${response.contentType};base64,${response.base64Content}`;
		this.base64Image = this.sanitizer.bypassSecurityTrustUrl(dataUrl);
	}
}
```

### Async pipe
```ts
imageData$ = this.fileService.getFileWithMetadata(123).pipe(
	map(response => {
		const dataUrl = `data:${response.contentType};base64,${response.base64Content}`;
		return {
			metadata: response,
			imageUrl: this.sanitizer.bypassSecurityTrustUrl(dataUrl)
		};
	})
)
```
```html
<!-- In your template -->
<ng-container *ngIf="imageData$ | async as data">
  <h2>{{data.metadata.name}}</h2>
  <p>Created by: {{data.metadata.author}} on {{data.metadata.creationDate | date}}</p>
  <img [src]="data.imageUrl" alt="{{data.metadata.description}}">
</ng-container>
```

## Notes
The `DomSanitizer` is a security service that helps prevent Cross-Site Scripting (XSS) attacks. 

When you create URLs dynamically (like creating object URLs from blobs) and try to use them in contexts like `<img src="...">`, Angular will block them as potentially malicious unless you explicitly mark them as safe. 

The sanitizer does this by:
1. Examining dynamic content before it is inserted into the DOM
2. Scrubs potentially dangerous content or blocks usage in sensitive contexts
3. Provides methods to explicitly mark content as trusted for specific contexts. 

<br>
Although base64 is easy to do, it increases payload size by 33%, resulting in slower initial page render for large files. It is best for smaller images (like icons or thumbnails). 

A better approach would be to use Blob URLs, which are more memory efficient, and better for larger files. However, they require proper cleanup (i.e. revoking URLs when done), and require sanitization. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTExNjUxODI2XX0=
-->