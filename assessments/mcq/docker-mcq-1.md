I will ask these questions in quiz format. Please convert them into an interactive quiz that asks me each question and evaluates my answers.

Q1) What is the main benefit of multi-stage Docker builds for Go applications?
A) They allow running multiple processes in the same container
B) They separate build dependencies from the final runtime image, producing smaller images
C) They automatically sign images for you
D) They enable container orchestration
Correct: B
Why correct: Multi-stage builds use a builder stage to compile the binary and copy only the result into a minimal runtime image, reducing image size and attack surface.
Why A is wrong: Multi-stage is about build separation, not multiple runtime processes.
Why C is wrong: Image signing is a separate step/tool (e.g., cosign), not provided by multi-stage builds.
Why D is wrong: Orchestration is outside the scope of Dockerfile staging.

Q2) Why is `CGO_ENABLED=0` often used when compiling Go binaries for containers?
A) To enable dynamic linking against glibc
B) To produce statically linked binaries that run in minimal images like `scratch` or `distroless`
C) To allow cross-compilation to Java runtimes
D) To reduce binary size by excluding symbols
Correct: B
Why correct: Disabling CGO produces a statically linked binary (no glibc dependency), allowing use of minimal base images.
Why A is wrong: `CGO_ENABLED=0` disables C bindings, not enabling glibc.
Why C is wrong: It doesn't relate to Java runtimes.
Why D is wrong: While it can affect size, the key reason is static linking for portability.

Q3) Which base image gives the smallest possible runtime for a fully static Go binary?
A) ubuntu:20.04
B) alpine
C) scratch
D) golang:latest
Correct: C
Why correct: `scratch` is an empty base image; when using a fully static binary, `scratch` yields the smallest image.
Why A is wrong: Ubuntu contains many packages and is large.
Why B is wrong: Alpine is small but still contains a shell/library; not minimal like `scratch`.
Why D is wrong: `golang:latest` is a builder image, not minimal runtime.

Q4) What is the risk of embedding secrets directly in Docker image layers?
A) Secrets get encrypted automatically in the registry
B) Secrets become part of image history and can be retrieved from layers
C) Secrets are automatically removed when the container starts
D) There is no risk if you use `--no-cache`
Correct: B
Why correct: Commands that add secrets create image layers containing them; anyone with image access can inspect layers to retrieve secrets.
Why A is wrong: Registries don't automatically encrypt layer contents in a way that prevents access.
Why C is wrong: Secrets in layers persist; containers don't remove them automatically.
Why D is wrong: `--no-cache` affects build caching, not layering of secrets.

Q5) Which Dockerfile instruction should you minimize to improve layer cache reuse?
A) ENV
B) RUN that does many package installs without combining commands
C) COPY
D) ENTRYPOINT
Correct: B
Why correct: Combining package installs into fewer RUN layers and ordering Dockerfile to maximize cache reuse reduces rebuild time; RUN commands create layers that change often if not combined.
Why A is wrong: ENV creates lightweight layers but is less often expensive; still relevant but not primary.
Why C is wrong: COPY changes often with source changes; but optimizing RUN order and combining reduces cache misses.
Why D is wrong: ENTRYPOINT is final and rarely causes cache churn.

Q6) What advantage does a distroless image provide over Alpine for Go apps?
A) Distroless includes a full package manager
B) Distroless reduces attack surface by omitting shells and package managers
C) Distroless enables dynamic linking by default
D) Distroless adds a GUI for debugging
Correct: B
Why correct: Distroless images contain only the runtime and necessary libs, omitting shells and package managers, reducing attack surface.
Why A is wrong: Distroless intentionally avoids package managers.
Why C is wrong: Distroless doesn't change linking behavior.
Why D is wrong: Distroless does not include GUIs.

Q7) Which practice improves reproducible builds for container images?
A) Using `latest` tags for base images
B) Pinning base image versions and using image digests (sha256)
C) Building images directly on developer laptops only
D) Avoiding CI entirely
Correct: B
Why correct: Pinning base images to specific versions or digests ensures consistent, reproducible image contents across builds.
Why A is wrong: `latest` is mutable and breaks reproducibility.
Why C is wrong: Developer laptops produce non-repeatable environments; CI ensures consistency.
Why D is wrong: CI is key for reproducible builds.

Q8) What is a good CI step to run after building a container image and before pushing to registry?
A) Immediately deploy to production
B) Run vulnerability scan and sign the image
C) Delete the image to save disk
D) Open a shell inside the container and edit files
Correct: B
Why correct: Scanning for vulnerabilities and signing images ensures security and provenance before pushing to registry.
Why A is wrong: Deploying without validation risks pushing vulnerable images.
Why C is wrong: Deleting image prevents pushing; not useful.
Why D is wrong: Editing image manually breaks reproducibility and automation.

Q9) How does a `.dockerignore` file improve build performance?
A) It includes extra files into the image for completeness
B) It reduces the build context sent to the daemon, avoiding copying irrelevant files into image layers
C) It compresses layers automatically
D) It runs faster CPU optimizations
Correct: B
Why correct: `.dockerignore` excludes files from the build context, shortening transfer time and preventing undesired files from being added to image layers.
Why A is wrong: It excludes, not includes.
Why C is wrong: It doesn't compress layers; it reduces context size.
Why D is wrong: It doesn't affect CPU optimizations.

Q10) Which approach helps ensure images are immutable and traceable in production deployments?
A) Tag images only with mutable tags like `latest`
B) Push images by digest (sha256) and record digests in deployments/GitOps manifests
C) Push the image and rebuild on each node
D) Share images only by FTP
Correct: B
Why correct: Using image digests guarantees immutability (the digest references an exact content) and enables traceability of deployed artifacts.
Why A is wrong: Mutable tags can change and break reproducibility.
Why C is wrong: Rebuilding on nodes undermines immutability and introduces variability.
Why D is wrong: FTP is insecure and not traceable in orchestration contexts.
