# Google Maps Autocomplete Integration

## Objective
Implement Google Maps Places Autocomplete with address selection, geocoding, and map display in a React application.

## Technology Stack
- React 18+ with TypeScript
- @react-google-maps/api library
- Google Maps JavaScript API
- Google Places API
- Tailwind CSS or styled-components

## Requirements

### 1. Google Maps API Setup
- Enable the following APIs in Google Cloud Console:
  - Maps JavaScript API
  - Places API
  - Geocoding API
- Secure API key with HTTP referrer restrictions
- Environment variable configuration

### 2. Core Features

**Address Autocomplete Input:**
- Real-time search suggestions as user types
- Debounced API calls to optimize requests
- Configurable search radius and country restrictions
- Display formatted address suggestions
- Support for custom styling

**Address Selection:**
- Parse selected place into structured address components:
  - Street number
  - Street name
  - City
  - State/Province
  - Postal code
  - Country
- Extract geographic coordinates (lat, lng)
- Store full formatted address

**Map Display:**
- Show selected location on interactive map
- Draggable marker to fine-tune location
- Update address when marker is moved
- Configurable zoom level
- Responsive map container

### 3. Components to Build

**PlaceAutocomplete Component:**
```typescript
interface PlaceAutocompleteProps {
  onPlaceSelect: (place: PlaceResult) => void;
  defaultValue?: string;
  placeholder?: string;
  countries?: string[]; // Restrict to specific countries
  types?: string[]; // 'address', 'establishment', 'geocode'
  bounds?: LatLngBounds; // Bias results to specific area
  className?: string;
}
```

**MapWithMarker Component:**
```typescript
interface MapWithMarkerProps {
  center: { lat: number; lng: number };
  zoom?: number;
  onMarkerDragEnd?: (location: { lat: number; lng: number }) => void;
  markerDraggable?: boolean;
  height?: string;
  className?: string;
}
```

**AddressForm Component:**
- Combines autocomplete input and map
- Pre-fills form fields with parsed address
- Validates address components
- Allows manual override of auto-filled fields

### 4. Hooks

**useGoogleMaps:**
```typescript
const { isLoaded, loadError } = useGoogleMaps();
```

**usePlacesAutocomplete:**
```typescript
const {
  suggestions,
  value,
  setValue,
  clearSuggestions
} = usePlacesAutocomplete({
  debounce: 300,
  requestOptions: {
    componentRestrictions: { country: 'us' }
  }
});
```

**useGeocoding:**
```typescript
const { geocodeAddress, reverseGeocode } = useGeocoding();
```

### 5. Utility Functions
- **parseAddressComponents**: Extract structured data from place result
- **formatAddress**: Create display string from address components
- **validateAddress**: Ensure required components exist
- **calculateDistance**: Distance between two coordinates
- **isWithinBounds**: Check if location is within bounds

### 6. File Structure
```
src/
├── components/
│   ├── maps/
│   │   ├── PlaceAutocomplete.tsx
│   │   ├── MapWithMarker.tsx
│   │   ├── AddressForm.tsx
│   │   └── GoogleMapsLoader.tsx
│   └── forms/
│       └── AddressInput.tsx
├── hooks/
│   ├── useGoogleMaps.ts
│   ├── usePlacesAutocomplete.ts
│   └── useGeocoding.ts
├── utils/
│   ├── addressParser.ts
│   ├── addressFormatter.ts
│   └── geocoding.ts
├── types/
│   └── maps.types.ts
└── config/
    └── maps.config.ts
```

### 7. Environment Variables
```
VITE_GOOGLE_MAPS_API_KEY=your_api_key_here
VITE_GOOGLE_MAPS_DEFAULT_CENTER_LAT=40.7128
VITE_GOOGLE_MAPS_DEFAULT_CENTER_LNG=-74.0060
VITE_GOOGLE_MAPS_DEFAULT_ZOOM=13
```

### 8. Advanced Features

**Smart Defaults:**
- Use browser geolocation to bias results to user's location
- Remember recently selected addresses
- Auto-detect user's country

**Validation:**
- Verify address is deliverable (for shipping)
- Check if coordinates are in valid service area
- Validate postal code format

**Performance:**
- Implement request throttling
- Cache geocoding results
- Lazy load Google Maps script
- Use session tokens to reduce API costs

**Accessibility:**
- Keyboard navigation for suggestions
- ARIA labels for screen readers
- Focus management
- High contrast mode support

### 9. Error Handling
- Handle API loading failures
- Show user-friendly error messages
- Fallback to manual address entry
- Rate limit error handling
- Invalid API key detection

### 10. Type Definitions
```typescript
interface PlaceResult {
  formattedAddress: string;
  addressComponents: {
    streetNumber?: string;
    street?: string;
    city?: string;
    state?: string;
    postalCode?: string;
    country?: string;
  };
  geometry: {
    location: {
      lat: number;
      lng: number;
    };
  };
  placeId: string;
}
```

## Deliverables
- Fully functional autocomplete component
- Interactive map with draggable marker
- Address parsing utilities
- TypeScript types for all components
- Responsive design
- Comprehensive error handling
- Usage documentation with examples
- API cost optimization strategies

## Testing Checklist
- [ ] Autocomplete shows suggestions as user types
- [ ] Selecting suggestion fills address fields correctly
- [ ] Map displays selected location accurately
- [ ] Dragging marker updates address
- [ ] Reverse geocoding works correctly
- [ ] Country restrictions filter results
- [ ] Debouncing reduces API calls
- [ ] Component works on mobile devices
- [ ] Keyboard navigation functions properly
- [ ] Error states display appropriately
- [ ] API key validation works
- [ ] Session tokens reduce costs
- [ ] Geolocation bias improves results
- [ ] Form validation prevents invalid addresses
- [ ] Accessibility features work correctly
