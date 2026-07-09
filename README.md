This is called a Multi-Stage Dockerfile. It uses two FROM instructions:

Builder stage – installs dependencies.
Final stage – creates the lightweight image that will actually run.
