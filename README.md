# go-wallhaven

A comprehensive Go wrapper for the [Wallhaven.cc](https://wallhaven.cc) API that provides easy access to wallpapers, user collections, and search functionality.

## Installation

```bash
go get github.com/davenicholson-xyz/go-wallhaven@latest
```

## Quick Start

### Basic Usage (Unauthenticated)

```go
package main

import (
    "fmt"
    "log"
    
    wapi "github.com/davenicholson-xyz/go-wallhaven/wallhavenapi"
)

func main() {
    // Create a new client
    client := wapi.New()
    
    // Search for wallpapers
    results, err := client.Search("nature").Get()
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Found %d wallpapers\n", len(results.Wallpapers))
    for _, wallpaper := range results.Wallpapers {
        fmt.Printf("ID: %s, Resolution: %s, Views: %d\n", 
            wallpaper.ID, wallpaper.Resolution, wallpaper.Views)
    }
}
```

### With API Key (Authenticated)

```go
package main

import (
    "fmt"
    "log"
    
    wapi "github.com/davenicholson-xyz/go-wallhaven/wallhavenapi"
)

func main() {
    // Create client with API key
    client := wapi.NewWithAPIKey("your-api-key-here")
    
    // Access user settings (requires authentication)
    settings, err := client.UserSettings()
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Thumbnail size: %s\n", settings.ThumbSize)
    fmt.Printf("Per page: %s\n", settings.PerPage)
}
```

## API Reference

### Creating a Client

#### `New()`
Creates an unauthenticated client for public API access.

```go
client := wapi.New()
```

#### `NewWithAPIKey(apikey string)`
Creates an authenticated client with your API key.

```go
client := wapi.NewWithAPIKey("your-api-key")
```

### Search Operations

#### `Search(query string)`
Search for wallpapers with a text query. Returns a `Query` object for chaining filters.

```go
// Basic search
results, err := client.Search("anime").Get()

// Search with filters
results, err := client.Search("landscape").
    Categories(wapi.General).
    Purity(wapi.SFW).
    MinimumResolution("1920x1080").
    Sort(wapi.Views).
    Order(wapi.Descending).
    Get()
```

#### `TopList()`
Get top-rated wallpapers. Use `Range()` to specify time period.

```go
results, err := client.TopList().Range(wapi.OneWeek).Get()
```

#### `Hot()`
Get currently trending wallpapers.

```go
results, err := client.Hot().Get()
```

### Individual Wallpaper

#### `Wallpaper(id string)`
Retrieve a specific wallpaper by ID.

```go
wallpaper, err := client.Wallpaper("6k3oox")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Title: %s\nResolution: %s\n", wallpaper.ID, wallpaper.Resolution)
```

### Filtering Options

#### Categories
Filter by wallpaper categories:

```go
// Single category
client.Categories(wapi.General)

// Multiple categories
client.Categories(wapi.General, wapi.Anime)
```

Available categories:
- `wapi.General` - General wallpapers
- `wapi.Anime` - Anime/manga wallpapers  
- `wapi.People` - People/photography

#### Purity Levels
Filter by content purity:

```go
// SFW only
client.Purity(wapi.SFW)

// SFW and Sketchy
client.Purity(wapi.SFW, wapi.Sketchy)

// All content (requires API key for NSFW)
client.Purity(wapi.SFW, wapi.Sketchy, wapi.NSFW)
```

#### Resolution Filtering

```go
// Minimum resolution
client.MinimumResolution("1920x1080")

// Specific resolutions
client.Resolutions("1920x1080", "2560x1440", "3840x2160")

// Aspect ratios
client.Ratios("16x9", "21x9")
```

#### Color Filtering

```go
// Filter by dominant color (hex without #)
client.Colors("ff0000") // Red wallpapers
```

#### Sorting and Ordering

```go
// Sort by different criteria
client.Sort(wapi.DateAdded)   // Newest first
client.Sort(wapi.Views)       // Most viewed
client.Sort(wapi.Favorites)   // Most favorited
client.Sort(wapi.Random)      // Random order
client.Sort(wapi.Toplist)     // Top rated
client.Sort(wapi.Hot)         // Currently trending

// Order direction
client.Order(wapi.Descending) // High to low
client.Order(wapi.Ascending)  // Low to high

// Time range for toplist
client.Range(wapi.OneDay)        // Past day
client.Range(wapi.ThreeDays)     // Past three days
client.Range(wapi.OneWeek)       // Past week
client.Range(wapi.OneMonth)      // Past month
client.Range(wapi.ThreeMonths)   // Past three month
client.Range(wapi.SixMonths)     // Past six month
client.Range(wapi.OneYear)       // Past year
```

### Pagination

#### `Get()`
Get the first page of results.

```go
results, err := client.Search("nature").Get()
```

#### `Page(pageNum int)`
Get a specific page of results.

```go
// Get page 2
results, err := client.Search("nature").Page(2)

// Iterate through pages
for page := 1; page <= 5; page++ {
    results, err := client.Search("nature").Page(page)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Page %d: %d wallpapers\n", page, len(results.Wallpapers))
    
    // Check if we've reached the last page
    if page >= results.Meta.LastPage {
        break
    }
}
```

### Method Chaining

All filter methods can be chained together:

```go
results, err := client.Search("cyberpunk").
    Categories(wapi.General, wapi.Anime).
    Purity(wapi.SFW).
    MinimumResolution("1920x1080").
    Colors("0066ff").
    Sort(wapi.Favorites).
    Order(wapi.Descending).
    Page(1)
```

### User Operations (Requires API Key)

#### `UserSettings()`
Get authenticated user's settings.

```go
settings, err := client.UserSettings()
```

#### `MyCollections()`
Get your personal collections.

```go
collections, err := client.MyCollections()
for _, collection := range collections {
    fmt.Printf("Collection: %s (%d wallpapers)\n", 
        collection.Label, collection.Count)
}
```

#### `Collections(username string)`
Get public collections for a specific user.

```go
collections, err := client.Collections("username")
```

### Tag Information

#### `Tag(id int)`
Get detailed information about a specific tag.

```go
tag, err := client.Tag(1)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Tag: %s (Category: %s)\n", tag.Name, tag.Category)
```


## Rate Limiting

Be respectful of the Wallhaven API rate limits:
- Unauthenticated: 45 requests per minute
- Authenticated: 200 requests per minute

## Examples

### Find 4K Gaming Wallpapers
```go
results, err := client.Search("gaming").
    Categories(wapi.General).
    MinimumResolution("3840x2160").
    Sort(wapi.Views).
    Order(wapi.Descending).
    Get()
```

### Random Anime Wallpapers
```go
results, err := client.Search("").
    Categories(wapi.Anime).
    Sort(wapi.Random).
    Get()
```

### Top Wallpapers This Week
```go
results, err := client.TopList().
    Range(wapi.OneWeek).
    Categories(wapi.General).
    Purity(wapi.SFW).
    Get()
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License 

## Acknowledgments

- [Wallhaven.cc](https://wallhaven.cc) for providing the API
