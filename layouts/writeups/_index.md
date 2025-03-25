{{ define "main" }}
  <h1>Writeups</h1>

  <!-- Liste horizontale des catégories -->
  {{ partial "categories.html" . }}

  <ul>
    {{ range where .Pages "Section" "writeups" }}
      <li>
        <a href="{{ .RelPermalink }}">
          {{ .Params.ctf }} : {{ .Title }} | 
          {{ with .Params.categories }}
            {{ . }}
          {{ else }}
            Misc
          {{ end }}
        </a>
      </li>
    {{ end }}
  </ul>
{{ end }}

