# Nob.h Cheatsheet 

#### Table of Contents
- [Miscellaneous](#misc)
- [Logging](#logging)
- [File Handling](#file-handling)
- [Dynamic Arrays](#dynamic-arrays)
- [String Builder](#string-builder)
- [String Helpers](#string-helpers)
- [Process Handling](#process-handling)
- [Nob Cmd](#nob-cmd)
- [Complation Helpers](#compilation-helpers)
- [Go Rebuild Urself Technology](#rebuild-urself)
- [Nob String View](#string-view)

## Miscellaneous <a name="misc"></a>

### Macros
```c
NOB_PRINTF_FORMAT(STRING_INDEX, FIRST_TO_CHECK)
NOB_UNUSED(value)
NOB_TODO(message)
NOB_UNREACHABLE(message)
NOB_ARRAY_LEN(array)
NOB_ARRAY_GET(array, index)
```

## Logging <a name="logging"></a>

### TypeDefs/Defines
```c
typedef enum {
    NOB_INFO,
    NOB_WARNING,
    NOB_ERROR,
    NOB_NO_LOGS,
} Nob_Log_Level;

extern Nob_Log_Level nob_minimal_log_level;
```

### Functions
```c
void nob_log(Nob_Log_Level level, const char *fmt, ...)
```

## File Handling <a name="file-handling"></a>

```c
typedef struct {
    const char **items;
    size_t count;
    size_t capacity;
} Nob_File_Paths;

typedef enum {
    NOB_FILE_REGULAR = 0,
    NOB_FILE_DIRECTORY,
    NOB_FILE_SYMLINK,
    NOB_FILE_OTHER,
} Nob_File_Type;

bool nob_mkdir_if_not_exists(const char *path);
bool nob_copy_file(const char *src_path, const char *dst_path);
bool nob_copy_directory_recursively(const char *src_path, const char *dst_path);
bool nob_read_entire_dir(const char *parent, Nob_File_Paths *children);
bool nob_write_entire_file(const char *path, const void *data, size_t size);
Nob_File_Type nob_get_file_type(const char *path);
bool nob_delete_file(const char *path);
const char *nob_path_name(const char *path);
bool nob_rename(const char *old_path, const char *new_path);
int nob_needs_rebuild(const char *output_path, const char **input_paths, size_t input_paths_count);
int nob_needs_rebuild1(const char *output_path, const char *input_path);
int nob_file_exists(const char *file_path);
const char *nob_get_current_dir_temp(void);
bool nob_set_current_dir(const char *path);
```

## Dynamic Arrays <a name="dynamic-arrays"></a>

### TypeDefs/Defines
```c
#define NOB_DA_INIT_CAP 256 // default value
```

### Macros
```c
nob_da_reserve(da, expected_capacity)
nob_da_append(da, item)
nob_da_free(da)
nob_da_append_many(da, new_items, new_items_count)
nob_da_resize(da, new_size);
nob_da_last(da)
nob_da_remove_unordered(da, i)
nob_da_foreach(Type, it, da)
```

## String Builder <a name="string-builder"></a>

### TypeDefs
```c
typedef struct {
    char *items;
    size_t count;
    size_t capacity;
} Nob_String_Builder;
```

### Functions
```c
bool nob_read_entire_file(const char *path, Nob_String_Builder *sb);
int nob_sb_appendf(Nob_String_Builder *sb, const char *fmt, ...) NOB_PRINTF_FORMAT(2, 3);
```

### Macros
```c
nob_sb_append_buf(sb, buf, size)
nob_sb_append_null(sb)
nob_sb_free_(sb)
```

## String Helpers <a name="string-helpers"></a>

### TypeDefs/Defines
```c
#define NOB_TEMP_CAPACITY (8*1024*1024)
```

### Functions
```c
char *nob_temp_strdup(const char *cstr);
void *nob_temp_alloc(size_t size);
char *nob_temp_sprintf(const char *format, ...) NOB_PRINTF_FORMAT(1, 2);
void nob_temp_reset(void);
size_t nob_temp_save(void);
void nob_temp_rewind(size_t checkpoint);
```

## Process Handling <a name="process-handling"></a>

### TypeDefs/Defines
```c
#ifdef _WIN32
typedef HANDLE Nob_Proc;
#define NOB_INVALID_PROC INVALID_HANDLE_VALUE
typedef HANDLE Nob_Fd;
#define NOB_INVALID_FD INVALID_HANDLE_VALUE
#else
typedef int Nob_Proc;
#define NOB_INVALID_PROC (-1)
typedef int Nob_Fd;
#define NOB_INVALID_FD (-1)
#endif // _WIN32

typedef struct {
    Nob_Proc *items;
    size_t count;
    size_t capacity;
} Nob_Procs;
```

### Functions
```c
Nob_Fd nob_fd_open_for_read(const char *path);
Nob_Fd nob_fd_open_for_write(const char *path);
void nob_fd_close(Nob_Fd fd);
bool nob_proc_wait(Nob_Proc proc);
bool nob_procs_wait(Nob_Procs procs);
bool nob_procs_wait_and_reset(Nob_Procs *procs);
bool nob_procs_append_with_flush(Nob_Procs *procs, Nob_Proc proc, size_t max_procs_count);
```

## Nob Cmd <a name="nob-cmd"></a>

### TypeDefs
```c
typedef struct {
    const char **items;
    size_t count;
    size_t capacity;
} Nob_Cmd;

typedef struct {
    Nob_Fd *fdin;
    Nob_Fd *fdout;
    Nob_Fd *fderr;
} Nob_Cmd_Redirect;
```

### Functions
```c
void nob_cmd_render(Nob_Cmd cmd, Nob_String_Builder *render);
Nob_Proc nob_cmd_run_async_and_reset(Nob_Cmd *cmd);
Nob_Proc nob_cmd_run_async_redirect(Nob_Cmd cmd, Nob_Cmd_Redirect redirect);
Nob_Proc nob_cmd_run_async_redirect_and_reset(Nob_Cmd *cmd, Nob_Cmd_Redirect redirect);
bool nob_cmd_run_sync(Nob_Cmd cmd);
bool nob_cmd_run_sync_and_reset(Nob_Cmd *cmd);
bool nob_cmd_run_sync_redirect(Nob_Cmd cmd, Nob_Cmd_Redirect redirect);
bool nob_cmd_run_sync_redirect_and_reset(Nob_Cmd *cmd, Nob_Cmd_Redirect redirect);
```

### Macros
```c
nob_cmd_append(cmd, ...)
nob_cmd_extend(cmd, other_cmd)
nob_cmd_free(cmd)
nob_cmd_run_async(cmd)
```

## Compliation Helpers <a name="compilation-helpers"></a>

### Macros
```c
nob_cc(cmd) // append compiler based on platform
nob_cc_flags(cmd) // appends -Wall and -Wextra
nob_cc_output(cmd, output_path)
nob_cc_inputs(cmd, ...)
```

## Go Rebuild Urself™ Technology <a name="rebuild-urself"></a>

### Functions
```c
void nob__go_rebuild_urself(int argc, char **argv, const char *source_path, ...);
```

### Macros 
```c
NOB_REBUILD_URSELF(binary_path, source_path)
NOB_GO_REBUILD_URSELF(argc, agv)
```

## Nob String View <a name="string-view"></a>

### TypeDefs/Defines
```c
typedef struct {
    size_t count;
    const char *data;
} Nob_String_View;
```

### Functions
```c
const char *nob_temp_sv_to_cstr(Nob_String_View sv);
Nob_String_View nob_sv_chop_by_delim(Nob_String_View *sv, char delim);
Nob_String_View nob_sv_chop_left(Nob_String_View *sv, size_t n);
Nob_String_View nob_sv_trim(Nob_String_View sv);
Nob_String_View nob_sv_trim_left(Nob_String_View sv);
Nob_String_View nob_sv_trim_right(Nob_String_View sv);
bool nob_sv_eq(Nob_String_View a, Nob_String_View b);
bool nob_sv_end_with(Nob_String_View sv, const char *cstr);
bool nob_sv_starts_with(Nob_String_View sv, Nob_String_View expected_prefix);
Nob_String_View nob_sv_from_cstr(const char *cstr);
Nob_String_View nob_sv_from_parts(const char *data, size_t count);
```

### Macros
```c
nob_sb_to_sv(sb)
SV_Fmt
Sv_Arg(sv)
```
